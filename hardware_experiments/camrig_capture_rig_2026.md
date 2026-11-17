# Camrig, scheduled capture rig and motion-track tuning (2026)

**Status:** In progress. A working Raspberry Pi 5 rig that records on a schedule, finds candidate insect tracks and uploads clips. The track analysis is the active work. Draft built from the repo and its README, so items marked TODO need results, photos or context from whoever ran the rig.
**Period:** June to October 2026
**Source:** https://github.com/SPAIA/camrig (private, 60 commits, branches `main` and `feature/versioned-detectors`). The repo includes two sample clips' labels and motion sidecars and three filter-search results.
**Related:** Builds on [event_camera_tests.md](event_camera_tests.md) and the camera comparison in the [Fraunhofer IZM study](../radar-feasibility-2025/README.md). Its data lands in the platform described in [../platform_experiments/edge_hosting_and_database_architectures.md](../platform_experiments/edge_hosting_and_database_architectures.md), and its tuning is covered in [../platform_experiments/data_context_validation_and_augmentation.md](../platform_experiments/data_context_validation_and_augmentation.md).

---

## Summary

- Goal. A repeatable test rig for tracking tiny moving objects, mainly insects, in video. It collects footage on a schedule, adds human counts as ground truth, and tunes the track detector against them.
- Hardware. A Raspberry Pi 5 with three interchangeable cameras and a clock that wakes it each morning.
- Method. Record intra-only video at 60 fps, find moving blobs against a slow background, link them into tracks, and filter out plants and shadows using hand labels.
- Result. In one 424 second labelled sample, tuned filters cut the candidate tracks from about 942 to about 54 per minute while keeping about 96% of the labelled insects. See "Results".
- Field lesson. Power. A Pi 5 under load browned out a battery rig in September 2026.

## Hardware

| Camera | Sensor | Why it is on the rig |
|---|---|---|
| Pi Global Shutter | Colour IMX296, 1456x1088, up to about 60 fps | Default. Global shutter avoids skew on fast insects, and colour might help identification |
| Basler ace 2 mono over GigE | Global shutter mono | Comparison. No colour filter, so more light and sharper pixels. Uses the Pi's Ethernet port, so internet moves to Wi-Fi |
| Pi Camera Module 3 | Colour IMX708, autofocus | Comparison. Focuses before each clip, then holds the lens position |

- Pi 5 with an NVMe drive for clip storage. Most rigs boot from a single NVMe with no SD card.
- The Pi 5 clock wakes the rig at 05:00 and it shuts down at 22:00. NTP sets the time at boot.
- Admin access over Tailscale. The only public path is the Cloudflare trigger described below.
- The Basler is probably the same family as the GigE camera in [event_camera_tests.md](event_camera_tests.md), which also notes the GigE limit of about 115 MB/s. TODO confirm the exact model.

## Capture

- A 5-minute clip every 30 minutes inside the 05:00 to 22:00 window (cadence changed on 7 Sep 2026).
- A remote start from a public page, so a person can count insects while the rig records.
- Three profiles. `mjpeg` (default) is intra-only, so every frame stands alone. `ffv1` is lossless but software encoding probably can't sustain 60 fps at full size. `raw` keeps Bayer frames for short fidelity tests.
- Settings chosen to protect small targets.
  - Intra-only video, because inter-frame codecs add motion compensation artefacts exactly where a tiny object moves.
  - Temporal denoise off, because it smears or erases insect-sized targets.
  - A 2 ms shutter to freeze motion, which needs good light. Gain is raised if it is too dark.
  - A timestamp for every frame, so velocities and count alignment are exact.
- This is also why a Pi 5 was chosen over a Pi 4 even though the Pi 4 has a hardware H.264 encoder. The rig doesn't want lossy video, and the Pi 5 has the storage bandwidth and a built-in clock.
- A focus page served over Tailscale shows a live view with a sharpness score and a rising tone as focus improves. A settings page edits the config. An optional Wi-Fi access point lets a phone reach the focus page when there is no network. It is off by default because of its power draw.

## Processing after capture

1. One low-priority ffmpeg job decodes each clip once. It writes a low-resolution H.264 preview and feeds grey frames to the motion analysis. The idle time between clips is far longer than the 2 to 4 minutes a clip needs.
2. The motion analysis (`blob-track-v1`) works like this.
   - Each frame is compared with a slow running-average background, not the previous frame. Frame-to-frame differences lose slow crawlers and split fast fliers into old and new positions.
   - Masks are added up over short windows of 6 frames, about 100 ms. A pixel must be active in a minimum number of frames, which drops single-frame sensor noise.
   - The result is reduced to a coarse 8 pixel grid and connected regions are labelled. Blobs link from window to window by nearest centre.
3. Each track gets four values meant to tell insects from plants. Straightness is net distance over path length. Chronic is how persistently its cells were active across the clip, since swaying vegetation re-uses the same cells. Step ratio is the largest hop over the typical hop. Footprint ratio is the area swept by the track over the average blob size.
4. The clip is uploaded to R2 as soon as its sidecars exist. Files are written under temporary names and renamed only when complete, and leftovers from a power cut are cleaned at boot.

## Labelling and filter tuning

- A click-to-label page marks each track as insect, other or unsure and appends it to a file beside the clip.
- A scoring command checks a filter setting against those labels in seconds.
- A random search over the filter thresholds picks the setting with the fewest non-insect survivors that still meets a recall target.
- A stitching pass merges track fragments that are probably one insect. In the sample clips about a third of the labelled insects were the same animal labelled twice.
- A path-matching module is the base for a second phase, which would also tune the motion analysis itself and so needs labels matched to tracks by shape.

## Results

Two labelled clips (29 Aug and 15 Sep 2026), 424 seconds in total, with 93 insect, 121 other and 34 unsure tracks.

| Run (20,000 trials, 22 Sep 2026) | Insect recall | Non-insect tracks kept | Candidate tracks per minute |
|---|---|---|---|
| No filters | 98.9% (92 of 93) | 94% (114 of 121) | 942 |
| Best, recall target at least 95% | 95.7% (45 of 47) | 32% (33 of 102) | 54 |
| Best, recall target at least 90% | 91.5% (43 of 47) | 31% (31 of 100) | 45 |

- The best thresholds use a minimum straightness of 0.07, a maximum chronic value of 0.21, a stitch window of up to 1.5 s and 0.078 of the frame width, and a burst filter that drops 24 or more simultaneous tracks. The maximum chronic value moved furthest from its unfiltered default (1.0 down to 0.21), so it may do the most work against swaying plants. TODO confirm by running each filter alone.
- The numbers come from the same two clips used to tune them, so they're an upper bound. TODO score a clip that wasn't used in the search.
- About 88% of the surviving candidates in the best run are still non-insect, because most blobs in the footage are background. The filters make review practical and don't yet give a clean insect count.
- TODO field results. Hours recorded, sites, insects per hour and comparison with the human counts from the trigger page.

## Lessons recorded in the repo

- **Power (7 Sep 2026).** A Pi 5 running the camera, GigE Basler, NVMe and Wi-Fi together spiked past what a cheap DC-DC converter and loose jumper wires could supply. The symptoms were under-voltage messages, then a watchdog reboot loop, then a brownout the watchdog couldn't recover from. The README's rules for a battery rig are to power the Pi through USB-C and not through GPIO jumper wires, to use a converter rated for a genuine 5 V and 5 A with overload protection, and to load-test with an inline meter while capturing and joining Wi-Fi at the same time.
- **Crash resilience.** The hardware watchdog is on, the log survives a hard reset, and the LED pulses every 2 seconds while the supervisor runs, so a dark LED means the program has hung. The README says this hides the symptom and isn't a fix for the power problem.
- **Boot and shutdown.** Neither does a bulk upload of pending clips, because that competes with recording and focusing on a freshly powered rig. An immediate upload after each clip is the default.
- **Cloudflare.** Uploads of readings gained a custom User-Agent and retries on 403, 408 and 429 on 2 Oct 2026.

## Open questions

- Field results and the comparison between human and rig counts.
- Which camera wins. The three cameras are on one rig for comparison, but no comparison is recorded.
- What the `feature/versioned-detectors` branch changes.
- Whether the rig can run on a battery. The README suggests an ESP32 companion that wakes the Pi on demand and notes that today's trigger assumes the Pi is on all day.
- Power draw for each camera and for a full day.

## Housekeeping

- Config and secret files (R2 credentials, device token) are kept out of the repo and only templates are committed. The repo does include sample clips' sidecars and previews.
- Licence: Apache 2.0.
