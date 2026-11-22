# Insect detection and trigger sensor experiments (2020 to 2026)

**Status:** Ongoing. Most approaches are dropped or parked. The camera rig is the current main track.
**Scope:** Every sensor and method SPAIA has tried to detect small insects and decide when to take a photo
**Documents:** One write-up per experiment, linked in the table below. The radar study with Fraunhofer IZM lives in its own folder, [radar-feasibility-2025](../radar-feasibility-2025/README.md). The backend, interface and data experiments that receive this data are in [platform_experiments](../platform_experiments/README.md).

---

## Summary

Since 2020 we have tried to answer one question. What is the cheapest, lowest-power way to know that an insect is in front of the camera? Cameras on their own saw too much noise and cost too much power. Cheap proximity and radar sensors either lacked the range, lacked the resolution or picked up too much of the environment. Most of the experiments ended for one of those reasons, and the current work goes back to cameras, with an event camera and a high-frame-rate camera.

## Experiments compared

| Period | Experiment | Platform | Approach | Outcome | Write-up |
|---|---|---|---|---|---|
| 2020 | Early camera detection | ESP32-CAM | Average brightness in rings, then cells, around the frame | **Dropped.** Averaging missed small insects, shadows triggered it, power use was too high | [early_camera_experiments.md](early_camera_experiments.md) |
| 2020 to 2021 | Raspberry Pi basic camera | Raspberry Pi 4, high-resolution camera | First background subtraction detector, after an abandoned PyTorch start | Evolved into the 2021 Pi detector. TODO results | [pi_basic_camera_2020.md](pi_basic_camera_2020.md) |
| 2021 | Raspberry Pi background subtraction | Raspberry Pi, Pi camera | Running-average background model, blobs of 1000 pixels and up | Superseded. TODO results and reason | [pi_background_subtraction_2021.md](pi_background_subtraction_2021.md) |
| 2021 to 2023 | MAX30102 | ESP32-CAM | Reflected IR from a heart rate sensor, as a trigger for the camera | **Dropped.** Detects movement on the bench, but the insect has to be very close | [max30102.md](max30102.md) |
| 2024 to 2025 | XIAO ESP32-S3 camera | ESP32-S3, camera, SD card | Background subtraction in the microcontroller, upload to an ingest Worker | Prototype stopped in Apr 2025. TODO results | [xiao_sense_esp32s3_2024.md](xiao_sense_esp32s3_2024.md) |
| 2024 | Acconeer A111 radar | Raspberry Pi | 60 GHz presence detector as a trigger | **Dropped.** Environmental noise | [ambassador_radar_and_pi_camera.md](ambassador_radar_and_pi_camera.md) |
| 2024 | Frame differencing | Raspberry Pi and Pi camera | Reference frame, contours of 20 to 1000 pixels | Still in the latest code. TODO results | [ambassador_radar_and_pi_camera.md](ambassador_radar_and_pi_camera.md) |
| 2024 to 2025 | ST VL53L7CX | Raspberry Pi | 8x8 zone time-of-flight distance sensor | **Dropped.** Resolution too coarse for smaller insects | [ambassador_radar_and_pi_camera.md](ambassador_radar_and_pi_camera.md) |
| 2025 | Radar feasibility with Fraunhofer IZM | Anechoic chamber | 60 GHz FMCW MIMO radar on 20 and 30 mm phantoms | Phantoms clearly detected. **Radar parked** as a later-phase sensor | [radar-feasibility-2025](../radar-feasibility-2025/README.md) |
| 2026 | Event camera | Raspberry Pi 5, Prophesee GenX320 | Event blobs with a nearest-neighbour tracker | **In progress.** TODO results | [event_camera_tests.md](event_camera_tests.md) |
| 2026 | Camrig capture rig | Raspberry Pi 5, three cameras | Scheduled 60 fps capture, motion tracks, filters tuned on labelled tracks | **In progress.** Candidate tracks per minute 942 to 54 at about 96% recall on the tuning clips | [camrig_capture_rig_2026.md](camrig_capture_rig_2026.md) |
| 2026 | Husqvarna biodiversity pilot proposal | Robotic mowers with a high-speed or event camera and radar | Mount sensors on mowers that stop at set spots | Submission for the Combient Foundry "Biodiversity with Automowers" call, deadline 10 May 2026. No outcome in the repo | None. The repo is a proposal workspace |

## What we learned

1. **Averaging hides small insects.** The 2020 camera tests and the 8x8 time-of-flight sensor both spread a small target over too many pixels or zones.
2. **Shadows trigger brightness-based detection.** This hit the 2020 camera tests, and the event camera notes expect the same problem.
3. **Proximity sensors need the insect close.** The MAX30102 worked on the bench and not in the field.
4. **Radar picks up the environment.** In the field the Acconeer radar was swamped by noise. The Fraunhofer study got clean detections only in an anechoic chamber, and did not test wind, vegetation or multipath.
5. **Power limits everything.** The Fraunhofer kickoff set a target of more than 2 years on a battery and a unit cost under €40. Continuous camera processing doesn't fit those numbers.

## Timeline

| Date | Event |
|---|---|
| Jan to Aug 2020 | Camera-only detection on the ESP32-CAM, in bands and then cells |
| Sep 2020 to Apr 2021 | Raspberry Pi basic camera, from a PyTorch start to background subtraction |
| Apr to Sep 2021 | Raspberry Pi detector with background subtraction, OLED and web controls |
| Oct 2021 | MAX30102 firmware v0.2 starts, with a fixed IR baseline |
| Apr 2022 | `field_agent` firmware starts, with a running average |
| Jun 2023 | Last `field_agent` commit |
| Apr 2024 | Raspberry Pi system with radar trigger, then camera motion detection |
| May 2024 | Time-of-flight experiments start |
| Sep 2024 | XIAO ESP32-S3 camera firmware starts. Last commit Apr 2025 |
| Sep 2024 | Radar tests with Fraunhofer IZM planned |
| Mar 2025 | Fraunhofer IZM final report delivered. Radar parked |
| Jul 2025 | Last commit on the `ambassador` repo |
| Oct 2025 to Mar 2026 | Sensor selection and camera measurements with Fraunhofer IZM |
| Apr 2026 | Event camera test repo starts |
| May 2026 | Husqvarna pilot proposal drafted |
| Jun to Oct 2026 | Camrig capture rig, from first commit to filter search and readings upload |

## Where things sit now

The main track is the camera rig, a Raspberry Pi 5 with a Basler camera and the GenX320 event camera as a second feed. A second rig, [camrig](camrig_capture_rig_2026.md), records on a schedule with three comparison cameras and tunes its track filters on labelled footage. Radar is parked. When it returns, the plan in the Fraunhofer write-up is to use camera detections as labelled ground truth and use radar to extend range past what the camera sees.

## Open questions

- Frame differencing results, ESP32 field data and the smallest insect each approach detected are missing from most write-ups. Each one has a TODO list.
- Why the detector changed from a fixed baseline to a running average in early 2022.
- Whether any of the trigger sensors still has a role, for example the MAX30102 in an enclosure the insect has to enter.
- How the event camera does outdoors, in sunlight, wind and moving shadows.

## Sources

- The source repos named in each write-up
- [radar-feasibility-2025/README.md](../radar-feasibility-2025/README.md) for the Fraunhofer IZM study, requirements and the Oct 2025 to Mar 2026 follow-up
