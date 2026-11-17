# Platform, data and interface experiments (2020 to 2026)

**Status:** Ongoing. Five generations of backend and six of user interface, and a first end-to-end loop from camera rig to human labels to filter tuning.
**Scope:** The software side of SPAIA's R&D. Where device data is hosted and stored, how people interact with devices and data, and how device data is checked and enriched with human input and outside sources
**Documents:** Three write-ups in this folder, plus the device experiments in [../hardware_experiments/README.md](../hardware_experiments/README.md) and the radar study in [../radar-feasibility-2025/README.md](../radar-feasibility-2025/README.md).

---

## Summary

SPAIA's devices only produce value if their data can be received cheaply, checked by people and set in context. These experiments tried different answers to three questions.

1. **Where does the data go?** Six hosting and database architectures, from FaunaDB behind Cloudflare Workers to a self-run Express and MariaDB API, to Postgres, and back to Workers with queues, R2 and D1.
2. **Who looks at it, and how?** An Angular dashboard for staff, then Svelte apps for device owners and the public, then a gamified counting app, with the framework, component library and map library each changing along the way.
3. **How do we know a detection is real, and what was happening around it?** Region labelling and verification, human counts aligned to camera frames, weather and habitat context from outside sources, and filter tuning on hand-labelled tracks.

## Write-ups

| Document | What it covers | Period |
|---|---|---|
| [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md) | Six architectures compared, the device upload paths and the problems recorded in the code | 2020 to 2026 |
| [user_interfaces.md](user_interfaces.md) | Marketing site, Angular dashboards, Svelte apps, the device commissioning wizard, simulators and the Bugmeister app | 2020 to 2026 |
| [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md) | Labelling, verification, human surveys, human counts as a reference, weather, AI habitat reads and filter tuning | 2021 to 2026 |
| [camrig_capture_rig_2026.md](../hardware_experiments/camrig_capture_rig_2026.md) | The capture rig that feeds the tuning work | 2026 |
| [xiao_sense_esp32s3_2024.md](../hardware_experiments/xiao_sense_esp32s3_2024.md) | ESP32-S3 camera firmware that uploads to the ingest Worker | 2024 to 2025 |

## Experiments compared

| Period | Experiment | Approach | Outcome |
|---|---|---|---|
| 2020 to 2021 | Edge API on FaunaDB | Cloudflare Worker with FaunaDB | Replaced by a relational API within months. TODO why |
| 2021 to 2024 | Self-run relational API | Express from an OpenAPI spec, Sequelize, MariaDB then Postgres | Ran for three years. Replaced by Workers in late 2024 |
| 2020 to 2021 | GraphQL | Apollo client in the dashboard, then a Workers GraphQL template | Not adopted |
| 2024 to 2025 | Edge ingest with queues | Hono Workers, R2, Cloudflare Queues, Postgres through Hyperdrive | Built and in use by later devices. TODO load results |
| 2024 | D1 for a public site | SvelteKit on Cloudflare with D1 and Stripe (pledge) | Built. TODO whether launched |
| 2026 | D1 citizen science app | SvelteKit on Workers, D1, R2, cron, better-auth | In progress, 35 migrations in six weeks |
| 2021 to 2024 | Labelling UI | Browser OpenCV guesses regions, a person labels and verifies | Built. TODO volume labelled |
| 2025 | Device commissioning wizard | Five steps, config file with an encrypted Wi-Fi password | Built. Key handling needs a decision |
| 2025 | Field survey form | Transect and timed counts with sightings and consent | Built. TODO use |
| 2026 | Human counts aligned to frames | Public trigger page, clock offsets, time-window device readings | Built. No comparison yet |
| 2026 | Weather and AI context | Bright Sky and Visual Crossing, vision model for plants and habitat | Built. Weather source replaced an AI guess |
| 2026 | Filter tuning on labels | Random search against 248 labelled tracks | Candidate tracks per minute 942 to 54, recall about 96% on the tuning clips |

## Repos reviewed

| Repo | Host | Period | Covered in |
|---|---|---|---|
| `0.02`, `field_agent` | Bitbucket, GitHub | 2021 to 2023 | [max30102.md](../hardware_experiments/max30102.md) |
| `camera-detection`, `mqqtDetection` | Bitbucket | 2020 | [early_camera_experiments.md](../hardware_experiments/early_camera_experiments.md) |
| `basic-camera`, `Scratch Version 0.001` | Bitbucket | 2020 to 2021 | [pi_basic_camera_2020.md](../hardware_experiments/pi_basic_camera_2020.md) |
| `components` | Bitbucket | 2021 | [pi_background_subtraction_2021.md](../hardware_experiments/pi_background_subtraction_2021.md) |
| `ambassador` | GitHub | 2024 to 2025 | [ambassador_radar_and_pi_camera.md](../hardware_experiments/ambassador_radar_and_pi_camera.md) |
| `sensor_tests` | GitHub | 2026 | [event_camera_tests.md](../hardware_experiments/event_camera_tests.md) |
| `camrig` | GitHub | 2026 | [camrig_capture_rig_2026.md](../hardware_experiments/camrig_capture_rig_2026.md) |
| `xiao_sense` | GitHub | 2024 to 2025 | [xiao_sense_esp32s3_2024.md](../hardware_experiments/xiao_sense_esp32s3_2024.md) |
| `SPAIA_WEB` (`spaia_web`) | Bitbucket | 2020 to 2021 | [user_interfaces.md](user_interfaces.md) |
| `Spaia dashboard`, `dashboard-2`, `spaia-dash` | Bitbucket, GitHub | 2020 to 2024 | [user_interfaces.md](user_interfaces.md), [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md) |
| `SPAIA API` (`spaia-api`) | Bitbucket | 2020 to 2021 | [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md) |
| `database`, `NODE API`, `api` | Bitbucket, GitHub | 2021 to 2024 | [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md) |
| `SPAIA_gaphql_server` | GitHub | 2022 | [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md) |
| `pledge` | GitHub | 2024 | [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md), [user_interfaces.md](user_interfaces.md) |
| `device_api`, `hono-api` | GitHub | 2024 to 2025 | [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md) |
| `lab` | GitHub | 2024 to 2025 | [user_interfaces.md](user_interfaces.md), [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md) |
| `app` (Bugmeister) | GitHub | 2026 | All three platform documents |
| `husqvarna` | GitHub | 2026 | A funding and partnership submission, not an experiment. It is mentioned in [../hardware_experiments/README.md](../hardware_experiments/README.md) |
| `docs`, `Documentation` | GitHub | 2025 to 2026 | `docs` is empty. `Documentation` holds these write-ups |

## Timeline

| Date | Event |
|---|---|
| Feb 2020 | Angular dashboard starts, on Firebase hosting |
| Apr 2020 | Hugo marketing site starts |
| Nov 2020 | Cloudflare Worker API with FaunaDB. GraphQL tried in the dashboard |
| Jun to Jul 2021 | MariaDB schema and OpenAPI Node API. Dashboard rebuilt as `dashboard-2`, then `spaia-dash` |
| Aug 2021 | Classification screen begins ("labels started") |
| Sep 2022 | Cloudflare Images service added to the Node API |
| Oct 2022 | GraphQL Worker template added and left |
| Feb 2024 | `pledge` starts on D1 |
| Apr 2024 | Data explorer with label and activity filters |
| May 2024 | API moves to Postgres. Last `spaia-dash` and `api` commits |
| Oct 2024 | `device_api` ingest Worker with queues |
| Nov to Dec 2024 | `hono-api` and `lab` start. Supabase replaces Firebase for new apps |
| May 2025 | Field observation form and verify endpoint |
| Jun 2026 | `camrig` starts |
| Aug to Oct 2026 | Bugmeister app built on D1. Rig readings API. Filter search on labelled tracks |

## What the repos show

- **Compute keeps moving to the edge, and relational storage stays.** The schema of events, regions and labels from 2021 survives into the 2024 Postgres move. Only the host changed.
- **Idempotent uploads matter.** Devices retry on weak connections, and the app has migrations to remove double counts and block out-of-order saves. The newest device endpoint is keyed so a retry can't count twice.
- **Provenance is built in late.** The 2026 tables keep a source column and the raw response for weather and plants, and the offset applied to a phone's clock. Older tables don't.
- **Gamification was cut back.** Rarity and XP were removed from the app's schema within weeks.
- **Labels are the scarce resource.** The tuning result depends on 248 labelled tracks from two clips.

## Open questions

- Why each architecture was replaced. Most repos stop without saying.
- Which interfaces and services are live today, with users and cost.
- A device-versus-human comparison. The pieces are in place and no analysis exists.
- Whether device events in Postgres and citizen observations in D1 should stay separate.
- Security and privacy. Consent handling for photos, the Wi-Fi password key, and secrets committed in older repos.

## Sources

- The source repos named above. Details and dates come from commit history, migrations, READMEs and code comments.
- [../radar-feasibility-2025/README.md](../radar-feasibility-2025/README.md) for the Fraunhofer IZM requirements that set the cost and battery targets
