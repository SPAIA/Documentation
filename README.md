# SPAIA R&D documentation

**Status:** Draft. Written from the source repos, so each document marks missing results and reasons as TODO.
**Scope:** The research and development behind SPAIA's insect monitoring devices and platform, from 2020 to 2026
**Maintainer:** Playstate UG (trading as SPAIA)

---

## Summary

SPAIA is trying to detect and count small insects cheaply and with little power, and to turn those detections into data that people can check and use. The work falls into three areas. What detects an insect, where the data goes and how people use it, and how a detection is judged to be real.

## Folders

| Folder | What it covers | Start here |
|---|---|---|
| [hardware_experiments](hardware_experiments/README.md) | Every sensor and method tried to detect insects and trigger a photo, with the outcome of each | [README.md](hardware_experiments/README.md) |
| [platform_experiments](platform_experiments/README.md) | Hosting and database architectures, user interfaces, and how device data is labelled, verified and enriched | [README.md](platform_experiments/README.md) |
| [field_deployment_2024](field_deployment_2024/README.md) | The 2024 Ambassador camera-trap season: four devices, 136,133 events, and the Haus Strobl summer run at Walenstadt | [README.md](field_deployment_2024/README.md) |
| [citizen_science_2026](citizen_science_2026/kraft_pit_vs_pankow_flower_bed.md) | Bugmeister hand counts from autumn 2026, comparing an unmanaged patch with an ornamental flower bed | [kraft_pit_vs_pankow_flower_bed.md](citizen_science_2026/kraft_pit_vs_pankow_flower_bed.md) |
| [radar-feasibility-2025](radar-feasibility-2025/README.md) | The 60 GHz radar feasibility study with Fraunhofer IZM, funded through IBB Transfer BONUS | [README.md](radar-feasibility-2025/README.md) |

## Reading order

1. [hardware_experiments/README.md](hardware_experiments/README.md) for the table of detection approaches and what we learned.
2. [radar-feasibility-2025/README.md](radar-feasibility-2025/README.md) for the one study run with an outside research partner.
3. [platform_experiments/README.md](platform_experiments/README.md) for the software side.
4. The individual write-ups for detail.

## Document format

Every write-up follows the same layout, so they can be compared.

- A header block with status, period, source repos and related documents.
- A summary, then the approach, results and what we learned.
- Open questions and sources at the end.
- Items marked TODO need results, numbers or context from whoever ran the work. They are the list of what to gather next.

## Handling notes

- The source repos are private. Several older ones have credentials committed, and each write-up lists them under Housekeeping. Rotate them before any repo is shared.
- The Fraunhofer IZM figures in `radar-feasibility-2025` come from IZM's report. Check with IZM before publishing that folder anywhere public.
- These documents describe what the code and commit history show. Where a reason for a decision isn't recorded, the document says so and does not guess.
