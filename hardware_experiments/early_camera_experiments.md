# Early camera-only detection experiments (2020)

**Status:** Finished. Dropped in August 2020 because brightness averaging missed small insects, shadows triggered it and it used too much power. Draft built from the source code, so items marked TODO need results, photos or context from whoever ran the tests.
**Period:** January to August 2020
**Source:** Private Bitbucket repo `playstate_africa/camera-detection`, 12 commits. Branches are `master`, `develop` and `render-cells`. `render-cells` has the cell-based `main.cpp` but not the headers and MQTT files it includes, so it can't build on its own. Two "added dev branch" commits on 5 Aug 2020 add those files to `develop`, which makes it the complete version of the cell approach.
**Also:** A second Bitbucket repo, `mqqtDetection` (one commit, 6 Aug 2020), holds only the MQTT helper and a sketch that publishes "hello" once a second. It is a connection test for the debugging stream described below.
**Related:** These experiments came before [max30102.md](max30102.md).

---

## Summary

- Goal. Detect an insect entering the camera's view using only the ESP32-CAM, with no extra sensor.
- Approach. Compare brightness between frames, first in rings around the frame edge, then in a grid of cells.
- Outcome. Averaging brightness could not reliably detect small insects, shadows set it off, and running the camera constantly used too much power. The detection code never got past a prototype. The first version prints INCOMING or OUTGOING to the serial port. The second streams raw cell values to a computer for debugging and has no working detector.
- What came next. A separate IR sensor, the MAX30102, in late 2021.

## Goal and idea

- We wanted to photograph insects without recording constantly.
- The idea was that an insect entering the frame changes brightness near the edge first, then further in. Watching rings from the outside inward would show both that something arrived and which way it moved.
- Detection had to run on the ESP32, which has little processing power and memory, so the method had to be simple.

## Hardware and tooling

- ESP32-CAM, AI Thinker pinout.
- Camera in grayscale mode, so each pixel is one brightness byte from 0 to 255.
- Flash LED on GPIO 4. The first version also sets up an LED on GPIO 14.
- PlatformIO with the Arduino framework. A Travis CI template is in the repo but nothing in it is enabled.
- TODO lens, enclosure, test setup, and what we pointed it at (live insects, printed targets, hand movement).

## Version 1, edge bands (25 to 27 Jan 2020, `master`)

How it works:

1. Capture a 320x240 grayscale frame (QVGA).
2. Split the frame into 5 nested rectangular rings, each 2 pixels wide. Ring 0 is the outer edge and ring 4 is the innermost.
3. Compute the average brightness of each ring.
4. Compare each ring's average with its value in the previous frame.
5. If the ring with the biggest change moved more than the threshold (2.2 brightness levels), look at where it is.
   - If it is next to the second-biggest change and further in than the last active ring, print INCOMING.
   - If it is further out than the last active ring, print OUTGOING.

Notes:

- The loop runs flat out with no sleep. Every pixel is visited once per ring, so it is a lot of work per frame. TODO frame rate we achieved.
- Commit messages trace the work. "band detection" and "not Working" on 25 Jan, "rendering bands" and "low res results" on 26 Jan, "rendering and double threshold" and "bands working" on 27 Jan, ending with "tweaking settings".
- The code prints band averages and previous values to the serial port for each detection, so it was tuned by reading logs. TODO any recordings or notes from those tests.

## Version 2, cells and off-device analysis (28 Jan to 5 Aug 2020, `develop`)

What changed:

- Rings became a grid of square cells, 4 pixels on a side.
- Frame size dropped to 160x120 (QQVGA), which gives a 40x30 grid of 1,200 cells.
- Each frame, the ESP32 averages the pixels in each cell and sends the 1,200 values over MQTT (TLS, port 8883) to the topic `camera/raw`.
- The device also subscribes to a `spaia/flash` topic, which looks like a way to switch the flash remotely.
- Wi-Fi and MQTT are set up in `setup()`, so this version needs a network connection to do anything.

Planned but not finished:

- A `Cell` class that keeps the last 5 values per cell, a movement direction (inward, outward or none) and its 8 neighbours. This would have allowed tracking motion across cells.
- A detector function, `findBugs()`, which is declared but never written.
- Constants for 3 detection bands and a cell size of 10 sit in `definitions.h` and don't match the 4-pixel cells and 3 bands used in `main.cpp`. TODO check which values were actually tested.

The MQTT stream was a debugging tool. It sent the cell values to a computer so we could visualise what the detector was seeing, instead of tuning it blind on the device. The branch name `render-cells` matches this. The receiving and visualisation code is not in this repo. TODO say where it lived, if it still exists.

## Time-lapse sketch (`scratch/SDTimeLapse.cpp`)

- A separate sketch that captures a photo at a fixed interval (set to 5000) and saves it to the SD card, with timestamps from NTP.
- Likely a way to collect real footage to test detection against, or a baseline "record everything" approach. TODO confirm.

## Results

Nothing in the repo records the results, but the team's conclusion was clear.

- Averaging brightness over a ring or a cell was not accurate enough to reliably detect smaller insects. A small insect changes only a few pixels, and averaging over a whole ring or a 4x4 cell dilutes that change.
- Shadows triggered the detector easily. A passing shadow changes brightness across a ring far more than an insect does, and the detector has no way to tell the two apart.
- Power was a concern. The camera and the per-pixel processing run continuously.

TODO add numbers and evidence if we have them.

- Smallest insect detected, and the size or distance where it stopped working.
- How often shadows caused false triggers, and in what conditions (cloud, sun angle, wind moving leaves).
- Frame rate and power draw.
- Example frames or logs.

## Why we moved on

Three problems, none of them fixable by tuning the threshold.

1. Sensitivity. Brightness averaging can't see small insects. A lower threshold would pick up small insects, but it would also pick up more noise.
2. Shadows. Brightness averaging responds to any change in light, so shadows cause false triggers. Fixing this would need something smarter than averaging, such as looking at shape or local contrast, which costs more processing and more power.
3. Power. Capturing and processing frames all the time doesn't suit a battery or solar device.

A separate trigger that reads infrared reflection avoids all three. It responds to something close to the sensor rather than to overall brightness, and it uses very little power. That led to the MAX30102 work in [max30102.md](max30102.md).

The commits stop in August 2020 and don't mention any of this, so it comes from the team rather than the code.

## What carried forward

A later camera-based attempt, with an event camera on a Raspberry Pi 5 in 2026, is in [event_camera_tests.md](event_camera_tests.md). It still has to deal with shadows.

- The ESP32-CAM as the platform, with PlatformIO.
- The idea of a separate, cheap trigger in front of a camera that stays off until needed. The MAX30102 firmware that followed in 2021 is built around it.
- The `spaia/flash` flash control and the SD card logging in the time-lapse sketch show up again in later firmware as flash handling and an SD card event log. TODO confirm the lineage.

## Notes on the repo

- The repo contains hard-coded Wi-Fi and MQTT credentials in the `develop` branch (`scratch/SDTimeLapse.cpp` and the MQTT header). If this repo is ever shared or published, rotate those first.
