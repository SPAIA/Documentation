# Context, validation and augmentation of device data (2021 to 2026)

**Status:** Ongoing. The newest work links a camera rig, human counts and filter tuning in one loop. Draft built from the repos, so results, sample sizes and the reasons for each change are marked TODO unless the code or a comment says why.
**Period:** 2021 to October 2026
**Source:** GitHub `SPAIA/spaia-dash`, `api`, `hono-api`, `lab`, `app`, `camrig` and Bitbucket `dashboard-2`, `database`, `node-api` (all private)
**Related:** [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md), [user_interfaces.md](user_interfaces.md), [camrig_capture_rig_2026.md](../hardware_experiments/camrig_capture_rig_2026.md)

---

## Summary

- Problem. A device reports that something moved. That is a weak claim. It might be an insect, a leaf or a shadow, and the event has no context such as weather, habitat or who checked it.
- Approach. Three layers added over five years. People label and verify what a device captured. People count insects at the same place and time, so device counts can be compared with a human reference. Outside data and AI add context, such as weather, plants and a habitat score.
- Result so far. One measured result, from the camera rig in September 2026. Filter settings tuned against about 250 hand-labelled tracks cut the candidate tracks the rig had to review from about 942 per minute to about 54 per minute while keeping about 96% of the insects. The rest is built but not evaluated. TODO.

## Layers compared

| Layer | Period | Where | What it adds |
|---|---|---|---|
| Region labelling | 2021 to 2024 | `spaia-dash`, Node API | A person draws a box on a detection and labels it |
| Event verification | 2021 to 2025 | Node API, `hono-api` | A person confirms an event, with a name and time |
| Field observations | May 2025 | `lab`, `hono-api` | Structured human surveys (transect and timed count) |
| Human counts as a reference | Jun 2026 onwards | `camrig`, `app` | Counts aligned to camera frames by clock |
| Weather | Sep 2026 | `app` | Station readings stored with each session |
| AI habitat and plant reading | Sep to Oct 2026 | `app` | Plants and habitat features read from a photo |
| Filter tuning on labels | Sep 2026 | `camrig` | Search for thresholds that keep insects and drop clutter |

## Region labelling and event verification

Data model (Jun 2021 onwards).

- An **event** is one device detection, with media and sensor data.
- A **region** is a box on an event image. Regions can have several labels, and labels can be shared across regions. A `labels` table holds a name and a count.
- An event can be **verified**. The API sets `verifiedAt` and `verifiedBy`, which records which logged-in person confirmed it and when. The same operation is a PATCH call in `hono-api` (May 2025).
- Event queries can filter on events that have or don't have regions or labels (Apr 2024), so a labeller can ask for events nobody has looked at yet.

Labelling screen (`spaia-dash`, from Aug 2021).

- It shows the event's images, flipping between them every 1.5 seconds so a person can spot what moved.
- It guesses where the insect is, with OpenCV in the browser. The two images are blurred with a 19x19 filter, subtracted, thresholded at 90 and traced for contours. Each contour becomes a candidate region saved to the API with no labels. The guess only runs for events that have no regions yet.
- The person can drag out their own region, move or delete one, and attach labels from a dialog.

How it fits the devices.

- The ESP32 firmware saved two photos per event, 50 ms apart, which is what the screen compares. This is why the firmware captures a pair (see [max30102.md](../hardware_experiments/max30102.md)).
- The v0.2 firmware wrote the first frame to both files, so its two images were identical and the difference would be empty. TODO check whether any v0.2 events were ever labelled.
- The older Pi detector sent bounding boxes with every event ([pi_background_subtraction_2021.md](../hardware_experiments/pi_background_subtraction_2021.md)). The classification screen could show those too.

Results. TODO. Useful numbers to add: events labelled, how many labels came from staff and how many from the public, how often the browser guess matched the final region, and the label vocabulary in use.

## Human surveys

`lab` and `hono-api` (2025) define a structured field observation.

- Two types, `transect` and `fit`. A FIT (flower-insect timed) count is a timed count at one flower or patch.
- Fields are time, location, weather, temperature, wind (calm, light, moderate, strong), season and a consent flag.
- Each observation has sightings with an insect group, an estimated count, behaviour, where it was seen, notes and an optional photo.
- The `lab` form walks a person through a transect step by step and asks them to photograph interesting sightings to help identification. Groups include bees, butterflies and moths, flies, beetles, ants and wasps.

## Human counts as a reference for devices

Two designs put a person's count next to a device's count.

**Camera rig trigger (`camrig`, Jun 2026).**

- A public page lets someone start a recording remotely while they count insects. Counts are stored in the cloud with a server timestamp. The rig reports the NTP-synced start time of its clip.
- To compare, the count time minus the clip start time gives an offset, and the matching frame is found in the clip's per-frame timestamp file. The design brief spells out the formula.
- The point is to get ground truth for a specific stretch of video. TODO how many sessions were counted this way.

**Bugmeister timed counts (`app`, Aug 2026).**

- A person's phone aligns its clock to server time before a session so their taps line up with a site camera. The applied offset is stored as `clock_offset_ms`, so a reader can see how far off the phone was and how far to trust the alignment (migration 0008).
- Counts happen at a spot, and a device mounted at that spot posts readings for time windows, so the two can be compared by spot and time.
- The app's cron job completes sessions that never finished, so abandoned counts don't hang around as open sessions.

TODO an actual comparison. Neither repo contains one yet.

## Weather and other context

- **Device sensors.** Temperature, humidity and pressure came from a BME280 or AHT20 on every device generation (see the hardware docs). Device readings in the app now store raw station pressure separately from sea-level pressure.
- **Public weather data.** For each session the app looks up a nearby station reading and stores the station, its distance, temperature, rain, wind, gusts, cloud cover, sunshine, humidity and pressure, plus the full raw response. It uses Bright Sky (German weather service open data) for Germany and Visual Crossing elsewhere (migrations 0024 and 0029). The UI shows four weather buckets and a windy flag.
- **A replaced approach.** The migration comment for the weather table says the reading is "sourced from Bright Sky instead of guessed by DeepSeek Vision off the spot photo". So an AI model first estimated the weather from a photo, and it was replaced with measured data. TODO why, for example accuracy or cost.
- **Location.** Geocoding and place IDs on spaces (migrations 0021 to 0023), with Turf used for areas and point-in-polygon checks.
- **Time.** An endpoint serves server time to phones for the clock alignment above.

## AI reading of habitat photos

Added Sep to Oct 2026 in the app.

- **Spot photos.** A vision model reads the photo and returns plants with a taxonomic rank (family, genus, species or a generic type) and habitat features in set categories (groundcover, soil, rock feature, woody debris, water, nesting feature, vegetation structure, other). The prompt asks for conservative, visible-only answers and says accuracy matters more than completeness. The result is stored on the spot as JSON.
- **Plant table.** Plants have a rank, an optional GBIF ID and names in several languages. Most rows begin as an AI guess with no GBIF match. The migration comment says a match is added later by hand or "once Pl@ntNet is wired up", so plant identification services are planned but not connected.
- **Habitat scan.** A page rates how insect-friendly a photo looks from 0 to 100 using the same model. Anonymous visitors get three scans, tracked in a cookie, before sign-up.
- **Provenance.** Tables keep a `source` column and the raw response, so AI guesses, public data and manual entries can be told apart and replaced later. The weather and plant tables follow the same pattern.

Results. TODO. No accuracy check against an expert survey is in the repos. Useful tests are plant matches against a botanist's list on the same spots, and habitat scores against the field count at those spots.

## Filter tuning on labelled tracks (`camrig`, Sep 2026)

The rig finds candidate insect tracks in each clip. Most are plants and shadows, so it needs filters.

- **Labels.** A click-to-label screen lets a person mark each track as insect, other or unsure. Labels are appended to a file next to the clip, with a normalised path, so they survive a change of analysis resolution.
- **Discriminators.** Straightness (net distance over path length, which is high for flight and low for swaying plants), how persistently a blob's cells stayed active, the largest jump against the typical step, the area swept by the track, track duration and a burst filter that drops many tracks appearing at once in similar directions.
- **Stitching.** A second pass merges fragments of one insect. In the two sample clips about a third of the labelled insects were the same animal labelled twice.
- **Search.** A random search over the filter thresholds, scored on recall of labelled insects first and non-insect survivors second. The author chose it over Optuna because a six to nine dimensional search that finishes in seconds doesn't need the dependency.
- **Dataset.** Two clips, 424 seconds in total, with 93 insect, 121 other and 34 unsure tracks.

| Run (20,000 trials) | Insect recall | Non-insect tracks kept | Candidate tracks per minute |
|---|---|---|---|
| No filters | 98.9% (92 of 93) | 94% (114 of 121) | 942 |
| Best with recall of at least 95% | 95.7% (45 of 47) | 32% (33 of 102) | 54 |
| Best with recall of at least 90% | 91.5% (43 of 47) | 31% (31 of 100) | 45 |

Reading the table.

- The insect total falls from 93 to 47 in the tuned runs. Those runs turn stitching on, so merged fragments are the likely cause. TODO confirm.
- The search and the scoring use the same two clips and labels. There is no held-out clip, so the numbers show what the method can do on this footage, not how it will do on new footage. The code names the next phase, which matches labels to tracks by path so thresholds that change the tracks themselves can be searched.
- The labelled tracks come from a person looking at the motion video. They measure agreement with a human, not a count of real insects. The human counts in the section above are the independent check.

## Open questions

- How many events have been labelled or verified, and by whom.
- How a device's count compares with a person's count in the same window. The pieces exist in `camrig` and `app` and no comparison does yet.
- Whether the AI habitat and plant reads are accurate enough to publish, and what the cost per photo is.
- Whether filter thresholds tuned on two clips hold across sites, seasons and lighting.
- Consent and privacy. The field observation form has a consent flag, but nothing says how photos with people in them are handled.
