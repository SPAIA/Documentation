# SPAIA database structures

**Status:** Schema from 2021 to 2022, defined in the `node-api` repo. No evidence it still runs. Draft built from migrations, models and seeders, so no row counts or production data.
**Period:** July 2021 to June 2022
**Source:** Private repo `playstate_africa/node-api`, folder `database/`. The 2020 dashboard used Firestore and is covered at the end.
**Related:** The API that reads and writes it is in [api_infrastructure.md](api_infrastructure.md).

---

## Summary

- Engine. MariaDB through Sequelize, with 22 migrations, 11 seeders and 14 models. Database name `spaia`.
- Shape. Three clusters of tables. Devices and who owns them, the events those devices produce, and the labelling of what is in each photo.
- Weak spots. Some migrations and models disagree, one table is legacy, and the species table is thinner than the spec says.

## Entity relationships

```
DeviceTypes 1---* Devices *---* (user, by Firebase uid) via UserDevices
     *                 1
     |                 |
DeviceType_Sensors     +---* SettingValues *---1 Settings
     |                 |
  Sensors *---1 SensorTypes
     1
     |
  SensorData *---1 Events *---1 Devices
                     1
                     +---* EventMedia
                     +---* Regions *---* Labels   (through Label_Regions)
```

## Devices and ownership

| Table | Columns | Notes |
|---|---|---|
| DeviceTypes | name, version | Seeded with Research Assistant (1) and Field Agent (2), both 0.0.1 |
| Devices | name, notes, ip, serial, typeId, createdBy, updatedBy, lastSeen | `name` is the human pairing name. `serial` is the chip ID. `lastSeen` changes on every authenticated device request |
| UserDevices | deviceId, user | `user` is a Firebase uid string. There is no users table. Many users can link to one device |
| Sensors | name, model, typeId, description | Seeded with the Bosch BME280 once per quantity, and the SHT20 for temperature and humidity |
| SensorTypes | name, description | Temperature, Humidity, Barometric pressure, UV |
| DeviceType_Sensors | deviceTypeId, sensorId | Which sensors a kind of device carries. Research Assistant gets the three BME280 rows, Field Agent gets the two SHT20 rows |
| Components | name, link, price, availability | A parts list with price and stock. Nothing in the API reads it |

Users are not stored. The Firebase uid goes into `UserDevices.user`, `Devices.createdBy` and `Events.verifiedBy`. The cost is that a user's name and email cannot be shown next to an event without asking Firebase.

## Events and readings

| Table | Columns | Notes |
|---|---|---|
| Events | time, type, deviceId, location, verifiedBy, verifiedAt, updatedBy | `type` is timer, manual or activity in the spec. `location` is a MariaDB POINT. `deviceId` is required |
| SensorData | sensorId, eventId, value | One row per reading, so an event with temperature and humidity has two. `value` is DECIMAL |
| EventMedia | eventId, fileId, source, filename | One row per photo. `filename` is what the device called it, `fileId` is the Cloudflare image ID, `source` is `cloudflare` |

This is a narrow table design for readings. A new sensor type needs no new column. The price is one join per reading when listing events.

## Labelling

| Table | Columns | Notes |
|---|---|---|
| Regions | eventId, x, y, w, h | A bounding box in pixels on the event's image |
| Labels | name, latinName, count | A species or category. Seeded with Spur-Throated Grasshopper and Praying Mantis |
| Label_Regions | labelId, regionId | Many to many, so one box can carry several labels |

`Labels.count` is stored, so it must be kept up to date by hand. Nothing in the API changes it. A count query on `Label_Regions` would be safer.

## Settings

| Table | Columns | Notes |
|---|---|---|
| Settings | name, type | A setting definition. Seeded with burst, brightness, exposure_bias, location and firmwareVersion |
| SettingValues | settingId, deviceId, value | Every change adds a row. The latest row per setting is the current value |

Values are strings. `type` says whether to read one as a number or boolean, but the API never converts. `Settings.type` is `null` as a string for `location`, which looks like a typo for the null type.

The query for current values uses a window function, `ROW_NUMBER() OVER (PARTITION BY settingId ORDER BY id DESC)`, and keeps rank 1. This needs MariaDB 10.2 or later.

## Other tables

- Vocabularies. A short word list with part of speech, 9 nouns seeded. Probably the source for three-word device names such as `lanky-band-brass`. The generator is not in the repo. TODO
- Device_Settings. The first attempt at settings, created in August 2021 and replaced by Settings and SettingValues. The table still exists and nothing uses it.
- SequelizeMeta. Migration history, managed by the CLI.

## Where the schema and the code disagree

| Item | Migration says | Model or spec says | Effect |
|---|---|---|---|
| `Events.verifiedAt` | STRING | DATE | Dates are stored as text |
| `Events.deviceId` delete rule | CASCADE, not null | SET NULL | Deleting a device deletes its events in practice |
| `Labels.latinName` | column exists | Missing from the model, present in the OpenAPI `Label` | API never returns it |
| `Sensors.typeId` | integer foreign key | STRING | Works by coercion |
| Region and Label_Regions delete rules | Changed by two later migrations | Model options repeat them | Fine today, easy to drift |

## Migrations, seeds and tooling

- `.sequelizerc` points the CLI at `database/config`, `models`, `migrations` and `seeders`.
- Timeline of the migrations. July 2021 for devices, events, regions, sensors and user links. August 2021 for labels, settings and vocabulary. September 2021 for the settings rewrite. October 2021 for event media. May 2022 for `filename` and `lastSeen`.
- Seed data is demo data. Two devices, `glitter-shell` (Research Assistant) and `demo-fieldAgent`, and two manual events dated 8 August 2021 at a point near Johannesburg.
- Only the `development` config is filled in. Test and production use `root` with no password and placeholder database names. TODO what the server ran with.

## The 2020 Firestore model

The dashboard in [dashboard_and_tagging.md](dashboard_and_tagging.md) did not use SQL. Its tagging page reads one Firestore collection, `events`.

- One document per photo. Fields the code uses are `url` and `rectangles`.
- `rectangles` is an array of `{x, y, width, height, title}`, embedded in the document.
- The document ID is the key for deleting.

That is simple to start with. It cannot say who drew a box, whether a human verified it, which device took the photo, or what the temperature was. Those gaps are what Events, Regions, Labels and SensorData were added to fix.

## Open questions

- How many events, regions and labels were ever stored, and were any from real field runs? TODO
- Is the committed `database/volume` a copy of the live database or a test one? TODO
- Should a users table exist, or is Firebase the permanent user store?
- If this moves to a hosted database, would PostGIS suit `Events.location` better than a MariaDB POINT?
