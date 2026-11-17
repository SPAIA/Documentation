# Ambassador, Raspberry Pi radar and camera trigger tests (2024 to 2025)

Status: first draft built from the source code. The repo has no notes or results, so anything about how well it worked is marked TODO.

Source: https://github.com/SPAIA/ambassador (private). Branches and dates:

| Branch | Dates | What it holds |
|---|---|---|
| `main` | 17 to 18 Apr 2024 | Timed photo logger plus the Acconeer radar SDK |
| `develop` | to 19 Apr 2024 | Radar triggers the camera over a local socket |
| `image-detection` | 20 Apr 2024 | Adds camera-based motion detection |
| `threads` | to 27 Jul 2025 | Latest work. Threaded logger and time-of-flight sensor experiments |

The `threads` branch is the most complete, so the descriptions below follow it unless they say otherwise.

This sits between [max30102.md](max30102.md) (ESP32 with an IR proximity trigger, 2021 to 2023) and [event_camera_tests.md](event_camera_tests.md) (event camera, 2026).

## 1. Summary

- Goal. Photograph insects with a Raspberry Pi and a good camera, using a cheap sensor or image analysis to decide when to take a photo.
- Three trigger ideas were tried in turn.
  1. A 60 GHz radar presence detector (Acconeer A111).
  2. Frame differencing on a low-resolution camera stream.
  3. An 8x8 time-of-flight distance sensor (ST VL53L7CX). Its resolution was too coarse for smaller insects.
- The code ends up on the camera approach. The radar listener is switched off in the latest `main.py`, and the time-of-flight work is a driver port with a single test program.
- Results. TODO. Nothing in the repo records how any of them performed.

## 2. Hardware

- Raspberry Pi, a 32-bit ARM target (`armv7l`). TODO model.
- Pi camera, driven with `picamera2`. The code mentions both the Pi HQ camera and Camera Module 3. It captures 2304x1296 stills (the Module 3 binned mode) and uses a 640x480 stream for detection. TODO which camera the field units used.
- Acconeer A111 radar on a SparkFun breakout, talking over SPI with GPIO through `libgpiod`. The board support file is `acc_board_rpi_sparkfun.c`.
- SparkFun Qwiic BME280 for temperature and humidity.
- ST VL53L7CX multizone time-of-flight sensor, in the experiments only.
- Power saving in the setup script. It turns off HDMI, Bluetooth, the Ethernet interface, Avahi and the status LEDs, and enables I2C and SPI. This suggests battery or solar use was planned. TODO confirm, and add measured power draw.

## 3. Stage 1, timed logger (`main` branch)

- Every 10 seconds the Pi takes a 1280x720 photo and reads temperature and humidity from the BME280.
- It appends a row to `data.csv`, uploads the CSV to the SPAIA API (`/field/events`) and the image to `/media`, then deletes both local files.
- There is no detection. It is a baseline that proves the capture, sensor and upload path. The ESP32 firmware uses the same two endpoints and the same CSV idea, so both device types feed the same backend.

## 4. Stage 2, radar trigger (`develop` branch)

How it works:

1. `radar/source/insect_detector.c` is a modified Acconeer presence detector example. It reads the radar 10 times a second.
2. When the detector reports a presence distance above zero, it opens a TCP connection to `localhost:9090` and sends the text `trigger_camera`. A commit message says the radar program waits for the Python side to answer before continuing.
3. The Python logger listens on that port and takes a photo on each message.
4. Each reading is also logged to `output.csv` with presence score, distance and a microsecond timestamp. The file is deleted every 100 readings, about every 10 seconds.

Radar settings, first version against latest:

| Setting | Early (`main`) | Latest (`threads`) |
|---|---|---|
| Start of range | 0.2 m | 0.1 m |
| Length of range | 1.4 m | 0.7 m |
| Receiver gain | default | 0.5 (default is 0.7) |
| Averaging (HWAAS) | 15 | 5 |
| Update rate | 10 Hz | 10 Hz |
| Detection threshold | 2.0 | 2.0 |
| Power save mode | sleep | sleep |

- The range shrank from 1.4 m to the first 0.8 m, which suggests the radar was most useful for things very close to it. TODO confirm.
- A code comment wonders whether to trigger on a presence score above 1100 instead of on distance. TODO say which one was used in the field.
- There's a test hook that fires the trigger at random about 30% of the time, commented out, for checking the socket.
- The radar sees movement in front of the sensor, not insects as such. TODO say what it reacted to in practice (insects, leaves, people, wind).

## 5. Stage 3, camera motion detection (`image-detection` and `threads`)

Added on 20 Apr 2024 ("basic motion detection in place") and refined in `threads`.

1. Capture 640x480 frames continuously.
2. Convert to grayscale and blur with a 21x21 Gaussian.
3. Subtract a stored reference frame and keep pixels that changed by more than 25 levels.
4. Dilate twice and find contours.
5. Accept a contour if its area is between 20 and 1000 pixels and its bounding box does not touch the frame edge.
6. Switch the camera to 2304x1296, take a still, switch back to low resolution, and reset the reference frame.
7. Store the contour in the CSV, upload the CSV and image, and delete both locally.

Other details:

- Capture runs on a thread, with a lock around the camera, and stops on Enter.
- Exposure compensation is raised by one stop "to ensure details in shadows". Sharpness is 2.0 and focus is manual at lens position 2.
- The small-area window of 20 to 1000 pixels is aimed at small objects. A helper script, `show_contour.py`, draws a sample contour of about 5x5 pixels on a 640x480 canvas, which suggests the detector was being tuned for very small blobs.
- The code reads an `IGNORE_ZONE` setting but doesn't appear to apply it in the detection loop.
- Compared with the 2020 ESP32 camera test, this compares to a fixed reference frame, not the previous frame, and works on contours, not brightness averages. TODO check whether shadows were still a problem, as in [early_camera_experiments.md](early_camera_experiments.md).
- `logger/mosock.py` is an earlier variant that runs motion detection and the radar socket listener together, at 1280x720 with a 200-pixel minimum area and a 30-second pause after each capture.
- `logger/motion.py` is a bare-bones test of the same algorithm that prints "bug" on detection.
- `requirements.txt` lists `ultralytics` and `supervision`, which are YOLO and detection tooling, but no code uses them. TODO say whether image classification was tried.

## 6. Stage 4, time-of-flight sensor experiments (`threads` branch, May 2024 and 2025)

- Goal. Use an ST VL53L7CX, which measures distance in an 8x8 grid of zones, to detect something moving in front of the camera. A change in distance in one zone would mean movement.
- Work done. The official ST Linux driver for the sensor is in `experiments/`, in both kernel-module and user-mode forms, with a device tree overlay and the sample programs.
- `experiments/spaia/spaia.c` is the project's own test program. It runs the sensor at 10 Hz in continuous mode, 8x8 resolution, 5 ms integration time, closest target first. For each zone it prints distance and the largest change seen so far.
- `experiments/vl53.py` is an unfinished Python attempt. It still has placeholder register names and doesn't run.
- `experiments/m3.py` steps the Camera Module 3 through manual focus positions 0 to 3.5 and saves a photo at each, to find the best focus for the target distance.
- The experiments folder starts with commits on 9 May 2024. The commits on 27 Jul 2025 add the VL53L7CX driver files and the test program, and remove a Python virtual environment that had been committed to the branch.
- Result. The resolution wasn't fine enough to detect smaller insects. The sensor splits its field of view into 64 zones, so each zone covers a wide patch of the scene, and a small insect moving within a zone barely changes that zone's single distance reading. TODO add the smallest insect it did detect, and the distance it was tested at.

## 7. Results

TODO. Please add whatever you remember or have saved.

- Detection range of the radar for insects, and what size and type of insect set it off.
- False triggers from each method (leaves, wind, shadows, people).
- Time from trigger to photo. The camera mode switch in stage 3 adds delay. TODO numbers.
- Photo quality and focus at the working distance.
- Power draw and run time.
- Smallest insect the time-of-flight sensor could see. It missed smaller insects because of its coarse 8x8 resolution (see section 6).

## 8. Observations from the code

- Triggering from the radar and from the camera ended up as two separate programs joined by a socket. The latest `main.py` has the radar listener commented out, so camera detection is the one that runs.
- Stage 3 trades power for flexibility. Streaming 640x480 and processing every frame keeps the Pi busy, which was the problem the 2020 ESP32 camera tests ran into. TODO confirm this was a concern here.
- Each capture switches the camera between two modes, which stops and restarts it. This probably costs the first moments of an insect visit. TODO measure.

## 9. Housekeeping

- The `main` branch has a `logger/.env` file with an API token committed. The `threads` branch replaces it with `.env.example`, but the token is still in history. Rotate it before sharing the repo.
