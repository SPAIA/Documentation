# Edge hosting and database architectures (2020 to 2026)

**Status:** Ongoing. The platform has been rebuilt several times and is now on Cloudflare Workers with D1 for the citizen science app and Postgres behind Hyperdrive for device data. Draft built from the repos, so reasons for each move are marked TODO unless the code says why.
**Period:** February 2020 to October 2026
**Source:** GitHub `SPAIA/api`, `SPAIA_gaphql_server`, `pledge`, `device_api`, `hono-api`, `lab`, `app`, `camrig` and Bitbucket `spaia-api`, `database`, `node-api`, `spaia-dashboard`, `spaia_web` (all private)
**Related:** [user_interfaces.md](user_interfaces.md), [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md). The devices that feed this backend are in [../hardware_experiments/README.md](../hardware_experiments/README.md).

---

## Summary

- Question. Where should device data land, and what should receive it? Devices are small, battery powered and often on weak connections, so the receiving end has to be cheap, always on and quick to answer.
- Answer so far. Six architectures in six years, from FaunaDB behind Cloudflare Workers, through a self-run Express and MariaDB API, to Postgres, and back to Workers with queues and D1.
- Direction. Compute moved to the edge in 2020 and 2022, retreated to a Node server from 2021 to 2024, and returned to Workers in late 2024. Storage moved from a document database to relational, and has stayed relational since.
- Results. Mostly TODO. The repos show what was built and when, and a few migration comments record real problems, which are included below.

## Architectures compared

| Stage | Period | Compute | Data store | Media | Auth | Device to cloud |
|---|---|---|---|---|---|---|
| 1. Edge API on FaunaDB | Nov 2020 to Apr 2021 | Cloudflare Worker (`spaia-api`) | FaunaDB | DigitalOcean Spaces | Auth0 and Firebase | Pi posts events, uploads photo to Spaces |
| 2. Self-run relational API | Jun 2021 to Apr 2024 | Express server from an OpenAPI spec (`node-api`, `api`) | MariaDB, then Postgres | Spaces, then Cloudflare Images | Firebase Admin | Multipart POST of events and photos |
| 3. GraphQL on Workers | Oct 2022 | Apollo on a Worker (`SPAIA_gaphql_server`) | None | None | None | Not used |
| 4. Edge D1 app | Feb to Nov 2024 | SvelteKit on Cloudflare (`pledge`) | D1 (SQLite) | None | Firebase Auth | Not a device path |
| 5. Edge ingest with queues | Oct 2024 to Jun 2025 | Hono Workers (`device_api`, `hono-api`) | Postgres through Hyperdrive | R2 | Firebase, then Supabase | `/upload` of CSV and JPEG, queue, parse, insert |
| 6. Edge app with D1 | Aug 2026 onwards | SvelteKit on Workers with a cron trigger (`app`) | D1 with 35 migrations | R2 | better-auth | JSON readings per time window, and a WebSocket from the rig |

## Stage 1, edge API on FaunaDB (2020 to 2021)

- `spaia-api` is a Cloudflare Worker built from the Workers starter template, with a router and a FaunaDB module. The `fauna` branch has "fauna working" (20 Nov 2020) and "switched to test index" (Jan 2021). Other branches are `add-fauna`, `headers` and `badversion`. `master` is empty.
- The Pi detector in [pi_basic_camera_2020.md](../hardware_experiments/pi_basic_camera_2020.md) wrote events straight to FaunaDB and images to DigitalOcean Spaces, so the first version had no API in between.
- The Angular dashboard in the same period used Firebase hosting, Auth0 and MQTT for live data (see [user_interfaces.md](user_interfaces.md)).
- TODO why FaunaDB was chosen and why it was dropped. The commit history stops in April 2021, two months before the relational schema appears.

## Stage 2, self-run relational API (2021 to 2024)

- **Database.** The `database` repo (30 Jun to 2 Jul 2021) is a MariaDB container with a dump. The tables are `citizens`, `citizens-roles`, `roles`, `devices`, `devices-users`, `deviceTypes`, `deviceTypes-sensors`, `sensors`, `sensorCategories`, `sensor-data`, `events`, `events-sensor-data`, `regions`, `labels` and `regions-labels`. An early commit is called "switched to citizens", so people were part of the model from the start.
- **API.** `node-api` (Bitbucket, Jun 2021 to Jun 2022) and `api` (GitHub, to May 2024) are one lineage. It is an Express server generated from an OpenAPI file, with Swagger docs and request validation. Sequelize migrations begin on 24 Jul 2021. The API has 14 paths covering devices, device registration and settings, events and event verification, regions, labels and images.
- **Media.** Photos went to Spaces first. A Cloudflare Images service was added in Sep 2022, and a media provider column was added on 30 Apr 2024, so both could coexist.
- **Logging.** Logtail through Winston.
- **Later work.** Projects and device projects (May to Aug 2023), activity and label filters (Apr 2024) and a switch to Postgres on 1 May 2024 ("switched to pg"). The final commit is 21 May 2024.
- TODO why MariaDB was replaced by Postgres, and where the API was hosted in production.

## Stage 3, GraphQL on Workers (2022)

- `SPAIA_gaphql_server` is one commit on 7 Oct 2022. It is the unchanged Cloudflare Workers GraphQL template, still with its Pokemon example schema and no SPAIA types.
- GraphQL was also tried earlier, in the `apollo` branch of the Angular dashboard in Nov 2020. That branch added Apollo client packages. The later `api-based` branch (Jul 2021) went the other way and removed them, using the REST API instead.
- Outcome. GraphQL was not adopted. TODO confirm why (REST was simpler for the devices, or the schema was too small to justify it).

## Stage 4, edge SQLite with D1 (2024)

- `pledge` is the first project on D1. It is a SvelteKit app on the Cloudflare adapter, with a D1 binding, Firebase Auth, Stripe and Mapbox. The schema has `habitats` (with a GeoJSON feature and area), `pledges`, `habitat_types` and join tables.
- A `wrangler-proxy` package lets local development reach the D1 database.
- It ran from 12 Feb to 14 Nov 2024 with 104 commits and branches for payments, profile, map animation and geocoding. TODO whether it launched.

## Stage 5, edge ingest with queues (2024 to 2025)

`device_api` (Oct 2024 to Apr 2025). The repo description says it is "lean and fast for our monitors to connect to in a hurry".

1. A device sends a CSV or JPEG to `POST /upload`.
2. The Worker stores the file in R2 and puts a message on a Cloudflare Queue (`csv-processing-queue`, batches of 10, 30 second timeout, 3 retries, dead-letter queue).
3. A queue consumer reads the file back from R2, parses the CSV (`timestamp,temperature,humidity,pressure,bboxes`) and inserts events and sensor data into Postgres through Hyperdrive. JPEGs become media rows.

The device only has to finish one upload and can go back to sleep. The parsing and database writes happen later and can retry. This is the endpoint shape that the XIAO firmware in [xiao_sense_esp32s3_2024.md](../hardware_experiments/xiao_sense_esp32s3_2024.md) and the ESP32 CSV format fit. TODO confirm the firmware's configured URL points here.

`hono-api` (Nov 2024 to Jun 2025). A second Worker for apps and dashboards.

- Hono with `zod-openapi`, so the OpenAPI document and Swagger page come from the same schemas that validate requests.
- Postgres through Hyperdrive, with R2 for images.
- Supabase JWTs checked with `jose`.
- Routes for devices, events, projects, images, uploads and field observations. Events can be verified with a PATCH call (May 2025).
- Pagination, sorting and date filters are shared schemas.

Other points.

- A `better-stack` branch in `device_api` appears to add hosted logging, going by its name and the "added logs" commits.
- Auth moved from Firebase (the Svelte dashboard in `device_api`) to Supabase (`lab` and `hono-api`) in Dec 2024.

## Stage 6, D1 app and rig links (2026)

`app` (Bugmeister, 25 Aug to 8 Oct 2026) is a SvelteKit app on Workers.

- **Database.** D1 through Kysely, with 35 numbered migrations in six weeks. They cover sessions, sightings, spots, spaces, habitat features, plants, weather, media, device readings and redeem codes.
- **Storage.** R2 buckets for avatars and media.
- **Metrics.** Workers Analytics Engine for events.
- **Scheduled work.** A cron trigger every 15 minutes completes counts whose device never sent the completion (a dead battery or a closed tab).
- **Auth.** The spec chose Supabase Auth with D1 for everything else. The code moved to better-auth in Sep 2026 (migration 0003), which keeps identity in D1 as well.
- **Domains.** Two custom domains on one Worker.

Field device link.

- `device_readings` (migration 0034) takes one row per upload covering a time window, with the device's tally and its raw track JSON stored as-is, so the format can change on the device without a migration.
- The unique key is the device and window start, so a retry after a lost response updates the row. It cannot count twice.
- On-device temperature, humidity and pressure were added a day later in the same style (0035).

`camrig` uses Cloudflare differently, as a way to reach a device that has no open ports.

- The Pi opens an outbound WebSocket to a Worker, which hands it to a Durable Object, one per rig, so a public page can start a recording.
- The public trigger is protected with Turnstile and rate limits. Counts go to D1. Admin access to the Pi stays on Tailscale.
- Clips and sidecars upload to R2 with `rclone`.
- On 2 Oct 2026 the rig started sending its own User-Agent and retrying on HTTP 403, 408 and 429, which suggests Cloudflare was rejecting some of its requests. See [camrig_capture_rig_2026.md](../hardware_experiments/camrig_capture_rig_2026.md).

## Problems recorded in the code

These come from comments and commit messages and are the most reliable evidence of what each architecture cost.

- **D1 foreign keys.** Migration 0029 explains that D1 enforces foreign keys per statement, so a table with live references can't be dropped and rebuilt. SQLite can't alter a CHECK constraint in place, so widening a weather source list meant copying the references aside, nulling them, rebuilding the table and restoring them.
- **At-least-once saves.** Migration 0027 removes duplicate sightings caused by resent batches, after "a request's response is lost". It backs up the rows it deletes. Later migrations (0030, 0031) replace per-tap requests with a whole-session snapshot, a revision counter and a per-session write token.
- **Public session IDs.** Session IDs are exposed on public share links, which is why the write token was added.
- **Workers and Postgres.** Both Worker APIs reach Postgres through Hyperdrive instead of opening connections themselves. TODO whether direct connections were tried first.
- **Cloudflare bot rules.** The rig's readings upload gained a custom User-Agent and retries on 403, 408 and 429 (2 Oct 2026). The commit doesn't say what failed, so TODO confirm.

## Open questions

- Why each stage ended. Only the Stage 3 and Stage 5 repos hint at it.
- Production hosting, cost and load for the Express API in Stage 2.
- Whether the D1 app and the Postgres APIs will merge, or stay as two stores (citizen observations in D1, device events in Postgres).
- Latency and cost of the queue-based ingest against posting directly to the API. No measurements in the repos.
- How many devices and events each stage handled.

## Housekeeping

- The Bitbucket `node-api` repo has a full MariaDB data folder committed (242 files, including the `mysql` system tables), plus log files and two sample CSVs. The GitHub `api` copy leaves these out. Treat the Bitbucket repo as sensitive and don't share it as is.
- The sample environment files in `api` and `node-api` hold short placeholder values, not real keys.
- The `app` Worker config includes a Cloudflare account ID and a D1 database ID. These aren't secrets on their own, but check them before publishing the repo.
- Secrets in the older device repos are listed in the hardware docs.
