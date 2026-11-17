# Event camera insect tracking tests (2026)

Status: first draft built from the code and readme in the repo. The repo has setup notes, scripts and a plan, but no results. Items marked TODO need results, photos or context from whoever ran the tests.

Source: https://github.com/SPAIA/sensor_tests (private). One real commit, 26 Apr 2026, on `master`. The `main` branch only holds a `.gitignore`.

This is the newest experiment. It comes after [early_camera_experiments.md](early_camera_experiments.md) (ESP32-CAM brightness averaging, 2020), [max30102.md](max30102.md) (IR proximity trigger, 2021 to 2023) and [ambassador_radar_and_pi_camera.md](ambassador_radar_and_pi_camera.md) (Raspberry Pi with radar, frame differencing and time-of-flight triggers, 2024 to 2025).

## 1. Summary

- Goal. Detect and track small moving insects using an event camera, which reports per-pixel brightness changes instead of full frames.
- Setup. A Raspberry Pi 5 with a Basler DART bcon MIPI camera that carries a Prophesee GenX320 event sensor and a Basler frame sensor.
- Method. Record raw event streams, then run blob detection and a simple tracker to draw coloured trails behind moving objects.
- Plan. Tune on ball bearings rolling in a tray, then move to live insects.
- Outcome. TODO. The repo describes the plan and the known problems but not what the tests showed.

## 2. Why an event camera

- It only outputs data where brightness changes. A still scene produces almost nothing, so data volume and processing load are small compared with full video. TODO confirm power draw, which we haven't measured.
- Time resolution is in microseconds, which suits fast insects such as bees and fruit flies.
- It looks like a fix for the 2020 camera problems. Detection works on motion at pixel level, so a small insect shouldn't be averaged away the way it was in a ring or a cell. The repo doesn't state this reasoning. TODO confirm it was the motivation.
- It does not escape the shadow problem. See section 7.

## 3. Hardware and software

- Raspberry Pi 5, reached remotely over Tailscale.
- Basler DART bcon MIPI CEP. One module with two sensors.

| Sensor | SDK | Use |
|---|---|---|
| Prophesee GenX320 (event) | Metavision / OpenEB | Insect detection and motion |
| Basler DART (frame) | Pylon | Colour stills and video |

- The two sensors need different device tree overlays and can't run at the same time. Switching means editing `/boot/firmware/config.txt` and rebooting. For event mode the `dtoverlay=basler-camera` lines must be absent. `startup.sh` loads the event overlays and activates the Python environment each session.
- Python with Flask, OpenCV and NumPy for the viewers and tracker.
- Mount, lens and lighting. TODO (the repo only describes the intended tray setup, see section 5).

## 4. Tools in the repo

| Script | Port | Purpose |
|---|---|---|
| `record_insects.py` | none | Records `.raw` (EVT3) files from the live camera, either one clip until Ctrl+C or a set number of fixed-length clips. Filenames carry a timestamp and an optional label such as the species. |
| `stream_metavision.py` | 5000 | Live preview in a browser. Has an optional noise filter, off by default, that keeps an event only if the same pixel and polarity fired within the last 10 ms. |
| `playback_server.py` | 5001 | Browse recordings and play them back at 1x to 50x speed. |
| `trail_server.py` | 5002 | Playback with detection, tracking and coloured trails. |
| `trail_live_server.py` | 5003 | The same detection and trails on the live camera. |
| `basler.py` | none | Records high-speed video from a Basler frame camera. See section 6. |
| `motion_glyphs.py` | none | Turns a video into images of motion trails. See section 6. |

## 5. Test plan

### Phase 1, ball bearings

- Shallow tray on a vibration-free surface, camera overhead and square to the tray floor.
- Ball bearings of different diameters, rolled by hand or by tilting the tray.
- White paper lining to give a diffuse background and cut reflections.
- Why ball bearings. They are round, so there's no orientation to worry about. Sizes are exact, which maps to pixel area at a known height. Their motion is predictable. The same tilt can be repeated to compare algorithm settings.
- Lighting notes in the readme.
  - Ring light around the lens is the best option for an overhead camera, because it keeps shadows under the object. Use DC-driven LEDs or PWM above about 20 kHz, since flicker creates false events.
  - A diffused LED strip around the tray edge gives better surface texture but throws strong shadows, so expect more false detections.
  - If specular glare stays a problem, put a polarising filter over the LEDs and a cross-polariser over the lens.

### Phase 2, live insects

| Species | Size | Movement | Expected problem |
|---|---|---|---|
| Fire beetles | 10 to 20 mm | Fast, erratic flight in short bursts | Fast, may leave the frame quickly |
| Ants | 2 to 5 mm | Steady walking on the ground | Low event rate, close to the noise floor |
| Wild bees | 10 to 15 mm | Flying, hovering, landing | Wing motion makes a dense cloud of events around the body |
| Fruit flies | 2 to 3 mm | Very fast, 3D flight | Close to the noise floor, may skip frames |

- Arena. Enclosed, to stop escapes and limit depth. Matte white or light grey floor. No transparent walls, because reflections and refraction add false edges. Keep temperature in mind, since cold insects move slowly and warm ones move fast and erratically.

## 6. Detection pipeline

The same code is in `trail_server.py` (recordings) and `trail_live_server.py` (live).

1. Group events into batches. Live uses about 33 ms, which is one batch per frame at 30 fps. Playback shortens the batch as speed goes up.
2. Count events per pixel in each batch and keep pixels with 2 or more events. This removes isolated noise.
3. Optionally dilate with an ellipse (default 3 px) to join events from the same object.
4. Find contours. Any contour larger than `max_area` is masked out and the contours are found again, so pieces of a big object don't show up as small blobs.
5. Keep contours between `min_area` (default 20 px) and `max_area` (default 400 px) as detections, using each one's centroid.
6. Match detections to existing tracks with a greedy nearest-neighbour rule. A match must be within 40 px. A track survives 8 frames without a match.
7. Draw a trail only after 5 consecutive matched frames, to suppress noise bursts. Trails are 45 points long by default.

All of these settings are adjustable in the browser, along with the accumulation time used to draw frames (default 2000 µs).

### Frame camera and glyph scripts

- `basler.py` records video from a Basler GigE camera using Pylon. It uses a 1024x768 region of interest in the middle of the 2448x2048 sensor, a 60 fps target, and a 2000 µs exposure to freeze fast motion. It buffers frames in RAM and writes an MP4 afterwards. The notes in the code give the GigE limit at about 115 MB/s, which allows roughly 116 fps at that region size. It is a different camera from the DART module in section 3, which is a MIPI part. TODO confirm which Basler camera this was run on.
- `motion_glyphs.py` takes an MP4, differences consecutive frames and accumulates the result with a decay of 0.90 per frame. Every 20 frames it saves a heat-map image where brighter pixels mean more recent or faster motion. Frame differencing starts from a blur and a change threshold of 8 levels.
- These give a way to compare event-camera detection with an ordinary high-speed video of the same movement. TODO say whether that comparison was done.

## 7. Known issues from the readme

Detection and tracking:

- Identity switching. When two objects pass close, the greedy matcher can swap their IDs.
- Track fragmentation. A fast object can skip pixels between frames, which splits one track into two short ones.
- Occlusion. A track hidden behind another object is dropped after 8 frames.
- Wing blur. A flying insect makes a dense event cloud that may exceed `max_area`, so the whole thing is filtered out.
- Stationary objects. An object that stops moving disappears from the event stream, so tracking only works while it moves.
- Scale. Area thresholds depend on camera height, so changing the mount invalidates tuned values.

Lighting and sensor:

- Shadow events. Moving shadows make almost as many events as the object. A ring light reduces this but doesn't remove it. Shadows also set off the 2020 camera detector.
- Specular reflections. Shiny ball bearings and insect wings produce bright flashes that look like dense event bursts.
- Vibration. A shaking mount drives events across the whole sensor at once.
- Hot pixels. Some pixels fire constantly and show up as one-pixel tracks. They can be masked by position.
- Vignetting. The event rate falls towards the frame edges, so objects there may fall below the threshold.

## 8. Results

TODO. Nothing in the repo records results. Useful things to add:

- Did the tracker follow ball bearings cleanly, and at what sizes and speeds?
- Did it detect and follow real insects, and which species?
- Smallest insect detected, and distance from the lens.
- Shadow and lighting behaviour compared with the readme's expectations.
- Recording samples and trail screenshots.
- Power draw of the Pi and camera in use.

## 9. Planned improvements

From the readme.

- Calibration. Record camera height and store a pixel-to-millimetre factor with each recording, so thresholds can be set in mm² and survive a change of mount.
- Hot pixel map. Record a dark frame at start-up and mask any pixel that fires too often.
- Sidecar file for each recording, with time, settings, species and lighting.
- Better tracking. Hungarian assignment in place of greedy matching, and a Kalman filter per track.
- Polarity. Track ON and OFF events separately, because their offset shows direction of motion.
- Export speed and acceleration per track to CSV.
- Wing and body separation. Find the dense wing cloud first, then the body as its density peak.
- Visualisation. Speed-coloured trails, longer persistent trails, an activity heat map, and auto-recording when a track is confirmed.
- Code. Move the duplicated tracker and blob detection into a shared module, and replace query-string defaults with a config file.

## 10. Open questions

- Does it work outside the lab, in sunlight and wind, where shadows and moving leaves create events? TODO
- Can it run on battery or solar? Power draw of a Pi 5 is far above the ESP32 setups. TODO
- Could the event camera act as a trigger for a normal camera, as the MAX30102 was meant to? TODO
