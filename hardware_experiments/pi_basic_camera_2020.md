# Raspberry Pi basic camera, first background subtraction detector (2020 to 2021)

**Status:** Superseded by the combined Pi detector in April 2021. The repo has no conclusion, so results and reasons are marked TODO. Draft built from the source code, so items marked TODO need results or context from whoever ran the tests.
**Period:** September 2020 to April 2021
**Source:** Private Bitbucket repo `playstate_africa/basic-camera` (30 commits, branch `master`)
**Also:** The Bitbucket repo `Scratch Version 0.001` shares this history. Its `master` stops at 10 Oct 2020, and it has branches `no-torch` (25 Nov 2020) and `version1` (30 Dec 2020), so the PyTorch decision can be dated.
**Related:** Starts a few months after [early_camera_experiments.md](early_camera_experiments.md). Direct predecessor of [pi_background_subtraction_2021.md](pi_background_subtraction_2021.md).

---

## Summary

- Goal. Run insect detection on a Raspberry Pi 4 with a camera, and store what it finds.
- Start point. PyTorch on the Pi, probably to run an object detection model. A PyCharm project file is named `yolo-5`. No model code is in the repo, and the PyTorch build is commented out by the end.
- What it became. OpenCV background subtraction on the live camera feed. When a new blob appears it saves a full-resolution photo with bounding boxes and weather readings, and uploads them to the cloud.
- Extras. A web page with a live video feed, BME280 readings and a control for the LED light, plus a timelapse mode that saves a photo every 20 minutes.
- Results. TODO. Nothing in the repo records how well it worked.

## Hardware

- Raspberry Pi 4, running a 64-bit balena image.
- Camera read through the V4L2 driver (`bcm2835-v4l2`). The code asks for 3200x2400, so it is a high-resolution module. TODO model of the camera.
- BME280 for temperature, humidity and pressure.
- NeoPixel LEDs on the Pi, used as a light. The code fills them with a near-white colour at start-up.
- Bluetooth scratch script that lists nearby devices. It isn't used by anything else. TODO what it was for.

## Timeline

| Date | Event |
|---|---|
| 29 to 30 Sep 2020 | Repo starts with a PyTorch build for the Pi 4. "Working pytorch and cam" on 30 Sep |
| 10 to 11 Oct 2020 | BME280 working, NeoPixels added |
| 12 to 15 Oct 2020 | Flask web server with a live video feed, JPEG capture. "Start motion" on 15 Oct, which is the first motion detection |
| 25 Oct 2020 | Background subtraction detector (`bg.py`) added. Events go to FaunaDB, which was added on 10 Oct |
| 2 Nov 2020 | Image upload to DigitalOcean Spaces added |
| 20 Nov 2020 | Wi-Fi connect removed ("bye bye wifi") |
| 25 Nov 2020 | Branch `no-torch` in the scratch copy points here, which marks PyTorch being dropped |
| 10 to 13 Dec 2020 | Refactoring. `SPAIA.py` replaces `bg.py` as the main script on 13 Dec |
| 30 to 31 Dec 2020 | Better resolution, and a working timelapse with the connection turned off |
| 5 Jan 2021 | Custom function in the main script |
| 10 Apr 2021 | Last commit. Moves to the combined detector in `components` |

## Detection

Three versions of the same idea sit in the repo.

| | `bg.py` (Oct to Nov 2020) | `SPAIA.py` (Dec 2020 to Apr 2021) |
|---|---|---|
| Background model | Running average, weight 0.01 | Running average, weight 0.1 |
| Smallest blob kept | 10,000 pixels | 1,000 pixels |
| Capture rule | When motion starts, after no motion | When the number of boxes goes up from the previous frame |
| Box merging | None | Merge boxes within 50 pixels |
| Frame size | Camera default | 3200x2400 requested |
| Offline | None | Events written to a local JSON file, uploaded on the next start |

Shared steps:

1. Read a frame, blur it with a median filter and a 5x5 Gaussian.
2. Update the background and subtract it from the frame.
3. Convert to grayscale, threshold at 30, dilate twice and find contours.
4. Keep large enough contours and take their bounding boxes.

Other files:

- `motion.py` is the standard "first frame" motion detector from an online tutorial, with a 500 pixel minimum. It was a starting point.
- `nmy.py` is a non-maximum suppression function for merging boxes. The main script doesn't use it and has its own merging.
- `camera.py` and `server.py` serve a live preview. `camera.py` still loads a leftover face detector file.

Capture and storage:

- Photos are saved as JPEG at quality 100, named by time, and uploaded to a DigitalOcean Spaces bucket.
- Each event is saved with its bounding boxes, time, file name and BME280 readings in FaunaDB, linked to a device record. A Firestore module is also in the repo but unused.
- `SPAIA.py` also writes a timelapse photo every 20 minutes (1,200,000 ms) to a separate folder.
- A Flask page shows the video feed, temperature and humidity, a still capture, and RGB control of the LEDs.

## Container setup

- Built with balena on a `raspberrypi4-64` image, privileged and on the host network, with the web server on port 80.
- OpenCV comes from the system packages. An older Dockerfile builds PyTorch and torchvision from source and starts a Jupyter notebook.
- GPU memory settings for the camera are in `balena.yml`.
- The readme only has notes for finding and killing the running Python process, and `v4l2-ctl` commands for white balance and manual exposure.

## What carried forward

The 2021 combined detector in [pi_background_subtraction_2021.md](pi_background_subtraction_2021.md) uses nearly the same code. It has the same denoise step, a weight of 0.1, a threshold of 30, two dilations, a 1000 pixel minimum and 50 pixel box merging. It adds the things this repo lacks.

- Web controls for exposure, brightness and resolution.
- An OLED screen, two buttons and a pause control.
- A rate limit on captures.
- The SPAIA API in place of FaunaDB.
- Wi-Fi set-up through a captive portal.

## Results

TODO. Useful things to add:

- Which insects it caught, and the smallest.
- False triggers, especially from shadows, wind and the moving background.
- What the 10,000 and then 1,000 pixel minimums were chosen for.
- Whether the timelapse was used to collect test footage.
- Photos from the units.

## Why we moved on

TODO. The detection idea carried straight into the 2021 work, so this reads as a stage in one project rather than a failed approach. The things it dropped were PyTorch and a model-based detector. TODO say why, for example speed on a Pi 4 or build effort.

## Open questions

- Whether YOLOv5 or any other model was actually run on the Pi, and how fast.
- Why the minimum blob size was lowered from 10,000 to 1,000 pixels.
- Which camera module needed a 3200x2400 request.

## Housekeeping

- `spaces.py` has a DigitalOcean Spaces access key and secret in plain text, `fauna.py` has a FaunaDB client secret, and `firestore.py` points to a Google credentials file. Rotate the keys before sharing the repo.
- A compiled Python cache folder and a few test photos are committed.
