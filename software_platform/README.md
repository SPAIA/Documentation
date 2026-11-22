# Data platform and interaction software (2020 to 2022)

**Status:** All four code bases are dormant. The last commit to the public website is August 2020, to the first dashboard November 2020, to the reviewer dashboard August 2021 and to the API June 2022.
**Scope:** The software that sits behind the sensor hardware. How a device registers, how its photos and readings reach a database, how people review them, and how the data is laid out.
**Documents:** One write-up per piece, linked below. The sensor experiments that feed this platform are in [hardware_experiments](../hardware_experiments/README.md).

---

## Summary

SPAIA built two generations of software around the insect cameras. The first was a browser dashboard that read live frames and photos straight from MQTT and Firestore. The second was a REST API with a relational database, meant to give devices a proper way to register and upload, and a new Angular dashboard, built at the same time, for reviewers to draw boxes and label what was captured.

Neither made it to a finished product. Together they show what the data model needs, which is a device, its readings, its photos, a box on each photo and a verified species name, tied to a person.

## Documents

| Document | Covers | Source repo | Period |
|---|---|---|---|
| [api_infrastructure.md](api_infrastructure.md) | Express and OpenAPI server, authentication, endpoints, the device upload path, hosting and known problems | `playstate_africa/node-api` | Jul 2021 to Jun 2022 |
| [database_structures.md](database_structures.md) | MariaDB tables, relationships, seeds, migrations, where schema and code disagree, and the earlier Firestore model | `node-api`, folder `database/` | Jul 2021 to Jun 2022 |
| [dashboard_and_tagging.md](dashboard_and_tagging.md) | Angular dashboard with a live MQTT camera view and a photo tagging page | `playstate_africa/spaia-dashboard` | Feb to Nov 2020 |
| [reviewer_dashboard.md](reviewer_dashboard.md) | Angular 12 client for the REST API with Firebase sign-in, device filtering and a box-editing review screen | `playstate_africa/dashboard-2` | Jul to Aug 2021 |
| [public_website.md](public_website.md) | Hugo landing page at spaia.co.za with a mailing list sign-up, and its Cloudflare deploy pipeline | `playstate_africa/spaia_web` | Apr to Aug 2020 |

## The two generations compared

| | Dashboard, 2020 | REST API and new dashboard, 2021 to 2022 |
|---|---|---|
| Client | Angular 9 web app | Angular 12 web app, `dashboard-2` |
| Login | Auth0 | Firebase Authentication |
| Storage | Firestore and DigitalOcean Spaces | MariaDB and Cloudflare Images |
| Live view | Yes, MQTT | No. The old debug camera route was not carried over |
| Device registration | None | Name then chip ID pairing |
| Settings per device | None | Append-only history, pulled by the device |
| Labels | Text in the photo document | Regions, labels and verification stamps |

The public website sits outside this picture. It is a static page that describes the project and collects email addresses, and it touches none of the data.

## How the pieces fit

```
Field device (ESP32-CAM)
   |  CSV + photos, device token           live frames over MQTT (2020)
   v                                              |
REST API (Express) --> MariaDB                    v
   |        |                            Dashboard /debug/camera
   |        +--> Cloudflare Images
   v
Reviewer dashboard (dashboard-2) --> regions, labels
```

## What we learned

1. **Specify first pays off.** The OpenAPI file gave us docs, validation and the list of work for free, and it is the part most worth reusing.
2. **Tags need an author and a place.** Text inside a photo record cannot be counted or audited. A region with labels and a verifier can.
3. **Devices need real credentials.** A chip ID as the only secret was fine for a bench, not for a deployment.
4. **Keep data out of git.** The database files, logs and a dev password were all committed.
5. **Uploads need to be confirmed.** The API answers before the data is saved, so a failed upload looks like a success.
6. **A generated client drifts.** The dashboard's API client was generated once, in August 2021, and the API changed after. Regenerating from the spec should be a script in the repo.

## Open questions

- Was the reviewer dashboard ever run against real field data? Its client was generated from an early spec and no longer matches the final API, so it needs regenerating before it can show a real photo.
- What replaced this stack after June 2022?
- Do the event camera and radar work from the hardware write-ups need the same event model, with different sensor types, or something new?
- Should a SPAIA platform start again from the schema, keeping the Events, Regions and Labels split, and drop the rest?

## Sources

- Private Bitbucket repos `playstate_africa/node-api`, `playstate_africa/spaia-dashboard`, `playstate_africa/dashboard-2` and `playstate_africa/spaia_web`, read in full for these write-ups
- Related write-ups in [hardware_experiments](../hardware_experiments/README.md), especially [early_camera_experiments.md](../hardware_experiments/early_camera_experiments.md) and [max30102.md](../hardware_experiments/max30102.md)
