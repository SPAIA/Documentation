# Raspberry Pi background subtraction detector (2021)

**Status:** Superseded. The repo has no conclusion, so the reasons are marked TODO. The last commit is 24 Sep 2021 and the MAX30102 firmware repo starts in Oct 2021. Draft built from the source code, so items marked TODO need results or context from whoever ran the tests.
**Period:** April to September 2021
**Source:** Private Bitbucket repo `playstate_africa/components` (50 commits, branches `master`, `threading`, `server-controls` and `v00.1`)
**Related:** Follows [pi_basic_camera_2020.md](pi_basic_camera_2020.md), where the same detector first appeared, and [early_camera_experiments.md](early_camera_experiments.md). The camera-detection idea returns in [ambassador_radar_and_pi_camera.md](ambassador_radar_and_pi_camera.md) and [event_camera_tests.md](event_camera_tests.md).

---

## Summary

- Goal. A self-contained Raspberry Pi insect detector. It watches the camera, saves a photo with the location of anything that moved, adds temperature, humidity and pressure, and uploads the result to the SPAIA API.
- Detection. OpenCV background subtraction on 640x480 video, run flat out in a background thread.
- Extras. An OLED screen, two buttons, an LED light, a web interface with a live video feed and camera controls, Wi-Fi setup through a captive portal, and a systemd service so it starts on boot.
- Results. TODO. Nothing in the repo records how well the detector worked.

## Hardware

- Raspberry Pi with a camera read through the V4L2 driver (the Dockerfile loads `bcm2835-v4l2`), so most likely a Pi camera module. TODO model of Pi and camera.
- Adafruit BME280 for temperature, humidity and pressure.
- SSD1306 128x64 OLED over I2C, for status messages.
- Two push buttons on GPIO 21 and 13.
- 4 NeoPixel LEDs on GPIO 18, used as a light for the camera.
- Power experiments. The repo started as a fork of the balena PiJuice UPS example, so the readme still describes that board. The Docker Compose file also lists a Waveshare UPS. A Waveshare e-paper display and a power button script sit in their own folders. TODO which power setup the field units used.

## Software layout

The work moved from separate balena containers to one combined program.

| Folder | What it is |
|---|---|
| `camera`, `oled`, `blinky`, `bme280`, `paper`, `power` | One folder per component, each built as its own balena container. Mostly April 2021 experiments |
| `combined` | All of the components merged into a single Python program. The readme runs this one as a systemd service called "SPAIA Insect detector" |
| `dashboard` | An Angular settings page, served from the Pi |
| `RaspiWiFi` | Submodule that gives a captive portal for entering Wi-Fi details |

Branches:

- `threading` (20 to 23 Apr). Moves detection onto its own thread.
- `server-controls` (28 to 29 Apr). Web controls, settings and a theme for the settings page.
- `v00.1` (to 19 Sep). A branch that looks like a first release, version 0.1.
- `master` (to 24 Sep). Adds button handling, the "play pause" control and fixes to events.

## Detection pipeline

From `combined/src/detector.py`.

1. Read a 640x480 frame with OpenCV.
2. Denoise with a median blur (5) and a 5x5 Gaussian blur.
3. Update a background model as a running average with a weight of 0.1 for the new frame, and subtract it from the frame. A slow-moving change in light is absorbed into the model, while a sudden change stands out.
4. Convert to grayscale, threshold at 30, dilate twice and find contours.
5. Keep contours with an area of at least 1000 pixels. A comment says this "makes sure motion area is reasonably small", but the code does the opposite and skips anything smaller than 1000.
6. Merge boxes that overlap or sit within about 50 pixels of each other.
7. If any boxes remain, call the capture handler.

Capture handler, from `combined/src/main.py`.

- Saves the frame, then rate-limits. The default config allows one capture (`burst = 1`) per 30 seconds (`capturelimit = 30`). Both come from the API settings.
- Builds an event with the boxes, time, device ID, image name, a fixed location, and temperature, humidity and pressure.
- If the device is online it posts the event to the API and uploads the image to DigitalOcean Spaces. If not it appends the event to a local JSON file and uploads it on the next start-up.
- At start-up it downloads settings from the API.

Controls.

- Button B pauses and resumes detection. Button A switches the LEDs on and off. Both together shut the Pi down.
- The web page serves a live MJPEG feed with the detection boxes drawn on, and routes for brightness, exposure bias (0 to 24, default 12), resolution and dynamic frame rate.
- Camera settings are changed by calling `v4l2-ctl`.

## Comparison with later detectors

| | Pi 2021 (this repo) | Pi 2024 ([ambassador](ambassador_radar_and_pi_camera.md)) |
|---|---|---|
| Reference | Running-average background, weight 0.1 | One stored reference frame, reset after each detection |
| Blob size accepted | 1000 pixels and up | 20 to 1000 pixels |
| Frame size | 640x480 | 640x480 for detection, 2304x1296 for the photo |
| Loop | Continuous, no sleep | Continuous |

- The 2021 detector only accepts large blobs, so it would have missed small insects. The 2024 version accepts much smaller ones, and `show_contour.py` there plots a contour of about 5x5 pixels.
- `camera/src/motion.py` is the standard "first frame" motion detector tutorial with a 500 pixel minimum, and the 2024 code follows the same steps.

## Results

TODO. Useful things to add:

- Which insects it caught, and the smallest.
- False triggers, especially from shadows, wind and the background model drifting.
- Power draw and battery life.
- Time from motion to saved photo.
- Photos and a unit in the field.

## Why we moved on

TODO. The commits stop on 24 Sep 2021, and the ESP32 and MAX30102 work starts in October 2021. If the move was driven by power or cost, it fits the later goals in the Fraunhofer study (under €40 per unit, more than 2 years on a battery). The code points the same way, since the detector never sleeps and runs a Pi at full load. Please confirm.

## Open questions

- Why the 1000 pixel minimum, and whether smaller thresholds were tried.
- Whether the units were deployed, and for how long.
- How much of this fed into the later Pi work in 2024.

## Housekeeping

- `combined/src/spaces.py` has a DigitalOcean Spaces access key and secret in plain text, and the readme has Wi-Fi network names and passwords. Rotate the keys and change the passwords before sharing the repo.
- The repo holds about 52,000 committed files from `node_modules` in the dashboard folder, which makes up most of its 69 MB of history.
