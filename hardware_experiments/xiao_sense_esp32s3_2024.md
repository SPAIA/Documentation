# XIAO Sense, ESP32-S3 camera with on-device motion detection (2024 to 2025)

**Status:** Prototype, last commit 27 Apr 2025. A working single-photo build, then a series of camera and memory fixes. Draft built from the repo and its guides, so results and the reason it stopped are marked TODO.
**Period:** September 2024 to April 2025
**Source:** https://github.com/SPAIA/xiao_sense (private, 46 commits). Branches `master`, `AHT20`, `bme280`, `cam_and_sd`, `cam_sd_and_wifi`, `i2c`, `initSequence`, `tuning` and `working-single-capture`.
**Related:** Follows [max30102.md](max30102.md) and the ESP32-CAM work. Uploads to the Worker described in [../platform_experiments/edge_hosting_and_database_architectures.md](../platform_experiments/edge_hosting_and_database_architectures.md).

---

## Summary

- Goal. A small battery-friendly camera trap on an ESP32-S3 that detects motion in the image itself, photographs what moved, logs temperature, humidity and pressure, and uploads over Wi-Fi.
- Board. A Seeed XIAO ESP32S3 Sense, going by the repo name and the ESP32-S3 target. A commit called "removing esp cam" (26 Sep 2024) suggests the work moved off the ESP32-CAM. TODO confirm the board and camera module.
- Why it matters. The MAX30102 trigger in [max30102.md](max30102.md) needed the insect very close to the sensor. This firmware goes back to image-based detection, as in the 2020 tests in [early_camera_experiments.md](early_camera_experiments.md), on newer hardware.
- Results. TODO. The repo holds a soak test ("2hour run", 23 Apr 2025) and many memory fixes, but no detection results.

## Architecture

An ESP-IDF project on FreeRTOS, with one component per job.

| Component | Job |
|---|---|
| `camera_interface` | Starts the camera, switches between a low-resolution detection mode and a high-resolution photo mode, saves JPEGs |
| `motion_detector` | Compares each frame with a background model |
| `aht_interface`, `bmp280`, `climate_interface`, `i2cdev` | Temperature and humidity from an AHT20 and pressure from a BMP280, on a shared I2C bus |
| `sdcard_interface` | Stores everything on an SD card in a dated folder layout, with CSV files for sensor data |
| `wifi_interface` | Connects, reconnects and reports status |
| `upload_manager`, `file_upload` | Queue files and send them over HTTPS with an API key |

- Upload has two modes. Real time (interval 0) uploads each file as soon as it exists. An interval mode queues files and sends them on a schedule. The default is real time. A proposal document describes how to wire the manager into the rest.
- The repo has a guide for each component and an overview written so that an AI coding assistant can pick up any file quickly.

## Motion detection

From the motion detector component and its guide.

1. Grab a frame in a low-resolution mode. The guide describes grayscale QVGA. The latest camera setup asks for YUV422 at SVGA, so the guide and the code have drifted. TODO check which is current.
2. Wait for 20 frames to build the background model, then keep it as a running average with a weight of 0.1.
3. Mark pixels that differ from the background by more than a threshold.
4. Label connected groups of changed pixels (8-way connectivity) and keep groups of at least 20 pixels.
5. Draw a box around each group, keep boxes between 30 and 10,000 pixels in area, and merge boxes that overlap by more than 40% (intersection over union).
6. When motion is found, switch the camera to a high-resolution mode, take a JPEG, and switch back.

This is the same family as the Pi detectors in [pi_basic_camera_2020.md](pi_basic_camera_2020.md) and [pi_background_subtraction_2021.md](pi_background_subtraction_2021.md), cut down to run in a microcontroller's memory.

## Problems and fixes

- **Heap fragmentation.** Switching resolution means shutting the camera down and starting it again, and each cycle fragments memory. The README records a patch to the camera driver, copied from another project, that keeps frame buffers allocated between shutdowns. The log has "basic working but memory issues on photos" (21 Apr 2025) followed by a week of camera restructuring ("camera alternative", "i2c alternative", "before re-init removal", "yuv244").
- **Colour format.** The latest commit ("yuv244", 27 Apr 2025) moves the camera to YUV422 and shrinks the detector by about a hundred lines. TODO confirm the aim was to save memory.
- **Camera swap.** "Switching cams" on 19 Nov 2024 and the camera configuration commits in Dec 2024 ("config camera breaking") show the sensor setup was changed more than once. TODO which modules were tried.
- **Clock.** A slower camera clock (10 MHz) is set to save power.
- **Soak test.** A two-hour run on 23 Apr 2025 changed Wi-Fi handling. TODO what failed.

## Wi-Fi set-up

- The Wi-Fi password in the device's config file can be stored encrypted, with an `ENC:` prefix and AES-CBC. The firmware decrypts it at start-up and copies a plain password through unchanged.
- The matching encryption runs in the browser, in the device commissioning wizard in `lab` (see [../platform_experiments/user_interfaces.md](../platform_experiments/user_interfaces.md)). The person types their Wi-Fi details, downloads `config.json` and copies it to the device.
- The key is a fixed value that sits in both the page source and the firmware, a comment in the page says so. That hides the password from anyone casually reading the file, but it is not strong protection, because anyone with the public page can read the key. TODO decide whether a per-device key or a pairing step is needed before units go to the public.

## Where it connects

- The repo's CSV matches the format the Worker in [../platform_experiments/edge_hosting_and_database_architectures.md](../platform_experiments/edge_hosting_and_database_architectures.md) expects, with a timestamp, temperature, humidity, pressure and bounding boxes. The Worker accepts CSV and JPEG uploads on one endpoint and processes them through a queue. The firmware's URL is configurable and I did not confirm it points there. TODO.
- The bounding boxes in the CSV are what the classification screen in [../platform_experiments/data_context_validation_and_augmentation.md](../platform_experiments/data_context_validation_and_augmentation.md) lets a person review.

## Results

TODO. Useful things to add.

- Smallest insect detected, and the distance.
- False triggers from shadows and wind, which were the main problem in the camera tests.
- Battery life and current draw in detection mode.
- Time from motion to saved photo, given the mode switch.
- Why work stopped in April 2025, and whether the Raspberry Pi route replaced it.

## Housekeeping

- The README still says "Add license information here".
- Wi-Fi and server settings come from the build config and a device config file. I found no committed Wi-Fi password or API key in the source. Check the build config before sharing the repo.
