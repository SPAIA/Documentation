# SPAIA REST API (node-api)

**Status:** Last commit June 2022. Working prototype that field devices uploaded to. Never hardened. Draft built from the repo, so items marked TODO need answers from whoever ran the server.
**Period:** July 2021 to June 2022, 63 commits
**Source:** Private repo `playstate_africa/node-api` on Bitbucket
**Related:** Data model in [database_structures.md](database_structures.md). The browser clients are in [reviewer_dashboard.md](reviewer_dashboard.md) and, for the 2020 version, [dashboard_and_tagging.md](dashboard_and_tagging.md). The firmware that calls this API is in [max30102.md](../hardware_experiments/max30102.md).

---

## Summary

- What it is. A Node.js and Express REST API that sits between the field devices, a user-facing app and a MariaDB database.
- What it does. Registers devices and links them to users, accepts event uploads (timestamp, sensor readings, photos) from devices, serves events back to users, stores bounding boxes and species labels on each photo, and hands each device its settings.
- Result in one line. The full device to database to reviewer path works. Security, error handling and tests were never finished, and several endpoints are stubs.

## Stack

| Part | Choice |
|---|---|
| Runtime | Node.js, Express 4 |
| API contract | OpenAPI 3.0.1 in `api/openapi.yaml`, scaffolded with OpenAPI Generator (`nodejs-express-server`) |
| Request validation | `express-openapi-validator`. Requests are validated, responses are not |
| Database access | Sequelize 6 over `mysql2`, with migrations and seeders |
| Database | MariaDB, run locally from the `mariadb:latest` Docker image |
| User auth | Firebase Authentication, ID tokens checked with `firebase-admin` |
| Device auth | Bearer token holding the device chip ID |
| Image storage | Cloudflare Images. DigitalOcean Spaces was used first |
| Logging | winston to console, `error.log`, `combined.log` and Logtail |
| Docs | Swagger UI at `/api-docs` |

The API-first approach shaped the code. The spec is the source of truth. Each operation in `openapi.yaml` names a controller through `x-eov-operation-handler`, the controller collects the validated parameters, and a service class does the work. Adding an endpoint means editing the YAML first.

## Request flow

1. `index.js` loads the environment and starts `ExpressServer` on `URL_PORT`.
2. `expressServer.js` adds CORS, body parsers, Swagger UI and the OpenAPI validator.
3. The validator checks the request against the spec and runs the security handler named on the operation.
4. It then calls the handler `controllers/<Name>Controller.js`.
5. `Controller.js` pulls path, query and body values into one object, with uploaded files saved under `uploaded_files/`. It calls the matching function in `services/`.
6. The service queries Sequelize and returns `Service.successResponse` or `Service.rejectResponse`.

`utils/openapiRouter.js` is still in the repo from the generator but nothing loads it. The validator's `operationHandlers` option does the routing.

## Authentication

Two schemes are defined in the spec and implemented in `adminService.js`.

- `firebase`. Used by anything a person does in an app. The server takes the bearer token, calls `admin.auth().verifyIdToken`, and keeps the uid and name. The Firebase project is the same one the 2020 dashboard used. The service account key is read from `keys/spaia.json`, which is git-ignored.
- `device`. Used by field hardware. The bearer token is the ESP32 chip ID. The server looks for a `Devices` row whose `serial` equals the token, and updates `lastSeen` if it finds one.

Problems found in the current code.

- The device check returns true for any non-empty token, even when no device matches. The lookup only fills in the device record. A missing device later causes an error instead of a 401.
- A chip ID is not a secret. Anyone who reads it off a board or sees it in traffic can upload as that device.
- The uid, name and device are stored on the `AdminService` singleton. Two overlapping requests can overwrite each other, so a user could get another user's device list. Per-request state belongs on `req`.
- Six operations have no security entry at all: `GET /device/ping/{serial}`, `PATCH /device/{id}`, `POST /device/{id}/setting`, `POST /media`, `POST /events` and `POST /image/{filename}`. The table below marks them.

## Endpoints

| Method and path | Auth | What it does |
|---|---|---|
| GET `/devices` | firebase | Devices linked to the caller, plus demo devices 1 and 2 for everyone, with type and sensors |
| POST `/devices` | firebase | Create a device and link it to the caller. Type is hard-coded to Field Agent |
| POST `/device/register` | firebase | Link an existing device, found by its name, to the caller. Optionally sets notes |
| POST `/link/device` | device | Device sends its human name, server stores the chip ID as `serial` |
| GET `/device/ping/{serial}` | none | Does a device with this serial exist |
| GET `/device/{deviceId}` | firebase | One device |
| PATCH `/device/{deviceId}` | none | Update device fields |
| GET `/device/{deviceId}/setting` | device | Latest value of each setting |
| POST `/device/{deviceId}/setting` | none | Append a new value for a setting |
| GET, POST, PUT `/me` | firebase | Profile. GET returns the name. POST and PUT echo the input and save nothing |
| POST `/field/events` | device | Multipart upload of a CSV of events plus images |
| POST `/media` | none | Upload one image, forward it to Cloudflare Images |
| GET `/events` | firebase | Events for one or more devices, with date range and paging, including media, sensor data, regions and labels |
| POST `/events` | none | Create events from JSON, with regions and sensor data in one call |
| PUT `/event/{eventId}` | firebase | Stub. Echoes the input |
| DELETE `/event/{eventId}` | firebase | Delete an event |
| PUT `/event/{eventId}/verify` | firebase | Stamp `verifiedAt` and `verifiedBy` |
| POST `/region`, PUT and DELETE `/region/{regionId}` | firebase | Create, change or delete a bounding box |
| GET, POST `/labels` | firebase | Search or create species labels |
| PUT `/label/{labelId}` | firebase | Stub. Echoes the input |
| POST, DELETE `/label/region/` | firebase | Attach or detach a label on a region |
| POST `/image/{filename}` | none | Declared, but `UtilsService` exports no `uploadImage`, so there is nothing behind it |

## How a field device uploads

This is the path the API supports, and the CSV format matches what the `field_agent` firmware writes. Pairing steps are inferred from the endpoints, TODO confirm against the firmware.

1. First boot. The user adds the device in an app, which creates or finds a `Devices` row with a three-word name such as `lanky-band-brass`. The device then calls `POST /link/device` with that name, and the server stores the chip ID as its serial. Name to serial is the whole pairing step.
2. Settings. The device calls `GET /device/{id}/setting` and applies what it gets back. Seeded settings are `burst`, `brightness`, `exposure_bias`, `location` and `firmwareVersion`.
3. Capture. The device writes a CSV to its SD card with the columns `temperature;humidity;time;media`, separated by semicolons, where `media` lists the image filenames.
4. Upload. `POST /field/events` sends the CSV and images as multipart. The server creates one `Events` row per CSV line, a `SensorData` row for temperature and one for humidity, and an `EventMedia` row per filename.
5. Image hosting. `POST /media` sends each image to Cloudflare Images. The server writes the returned image ID into the `EventMedia` row whose filename matches, then deletes the local file.

Things to know about steps 4 and 5.

- Temperature and humidity are matched to sensor IDs 1 and 2 by number, not by looking up which sensor the device has. A device with different sensors would be mislabelled.
- `uploadMedia` resolves its promise before Cloudflare answers. The device gets a 200 even when the upload later fails, and the failure only shows in the logs.
- Matching a Cloudflare ID back to a database row depends on the filename. It splits on `_` and `.jpg`, and the controller appends `_<timestamp>` to uploads, so the match is fragile.
- Rows are created inside a streaming CSV parser without waiting for the inserts. The response goes out before the data is written, and a bad row cannot fail the request.

## Review and labelling flow

This is the server-side version of the 2020 Firestore tagger described in [dashboard_and_tagging.md](dashboard_and_tagging.md). The client for it is the reviewer dashboard in [reviewer_dashboard.md](reviewer_dashboard.md), which uses only part of it.

1. `GET /events?deviceId=...` returns events newest first, each with its image IDs, readings and any regions.
2. The reviewer draws a box. `POST /region` stores x, y, w and h as integers against the event.
3. The reviewer picks a species. `POST /labels` creates it if needed and `POST /label/region/` attaches it. One region can carry several labels.
4. `PUT /event/{id}/verify` marks the event as checked by that user.

Seed data has two labels with Latin names. The Latin name column exists in the migration but not in the Sequelize `Label` model, so the API never returns it. The OpenAPI `Label` schema lists it.

## Deployment and operations

- Hosted on a single server. `.vscode/sftp.json` uploads the project over SFTP to `node-api/` on that host, with upload-on-save turned off. The spec lists the dev URL as `https://api.spaia.co.za`.
- No Dockerfile or process manager config. `npm start` runs nodemon, and `prestart` runs `npm install`. TODO how it was kept alive on the server.
- The Docker script in `package.json` starts MariaDB with a bind mount at `database/volume`, which is what ended up committed. See the section below.
- Logs go to winston transports and to Logtail, added March 2022. `LOGTAIL_TOKEN` and the Cloudflare token come from `.env`, which is git-ignored.
- `npm run refresh` drops, creates, migrates and seeds the database in one go. It is the quickest way to rebuild a dev database.
- No tests. The mocha and chai packages are installed but no test files exist.

## Housekeeping issues in the repo

- `database/volume/` holds the whole MariaDB data directory, 24 MB and 242 files, including the system `mysql` tables and the `spaia` schema as `.ibd` files. It should be git-ignored and removed from history.
- `combined.log` and `error.log` are committed. `combined.log` has about 2,500 lines, mostly startup messages, and 189 error lines.
- `database/config/config.json` and the `docker` script hold a development database password in plain text. It looks like a local dev value. TODO confirm it was never used on the server.
- `.vscode/sftp.json` holds the server address and login name. No keys appear in it, but it points at production.
- `env.default` has placeholder values only.
- `GET /device/{id}/setting` builds its SQL by joining the device ID into a string. The validator makes the ID an integer, so it is not exploitable today, but a bind parameter is the right fix.
- The `object` query flag on the settings endpoint never works. The controller passes `object`, and the service reads `isObject`.
- `eventId` and `regionId` have cascade rules in migrations. The Sequelize models declare different ones. See the schema doc.

## What I would keep and what I would change

Keep.
- The OpenAPI-first layout. It gave us docs, validation and a clear list of operations for free.
- Pairing by name then serial. It needs no QR code or provisioning step in the field.
- Settings as an append-only history, with a query for the latest value. We can see when a setting changed and roll back.
- Regions and labels as separate tables. One photo can hold several insects, and the same species can be reused.

Change.
- Authenticate devices with a per-device secret, generated at pairing, not the chip ID.
- Put request state on `req`, not on a shared object.
- Add security to the six open operations, or remove them.
- Return from uploads only after the data is saved, and make the CSV import one transaction.
- Replace the filename match with an ID the device sends in the CSV.
- Move the database out of git.

## Open questions

- Did the reviewer dashboard ([reviewer_dashboard.md](reviewer_dashboard.md)) ever run against this server? Its generated client points at `localhost:8080` and predates most of the endpoints. TODO
- Was it ever in production use with real field data, and how many events are in it? TODO
- Did the Pi detector (see [pi_background_subtraction_2021.md](../hardware_experiments/pi_background_subtraction_2021.md)) use it, or only the ESP32 firmware? TODO
- What replaced it after June 2022? TODO
