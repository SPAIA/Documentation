# Citizen science counts, autumn 2026: Kiez Café Kraft Pit vs Pankow flower bed

**Status:** Draft. Data pulled from the production `bugmeister` D1 database on 2026-11-23.
**Period:** 2026-09-06 to 2026-10-14
**Source:** `bugmeister` D1 (Cloudflare), tables `sessions`, `sightings`, `weather_observations`, `session_creatures`, `plant_observations`, `habitat_features`, `eval_items`
**Related:** [platform_experiments/spaia_fixed_camera_habitat_change_experiment.md](../platform_experiments/spaia_fixed_camera_habitat_change_experiment.md), [software_platform/database_structures.md](../software_platform/database_structures.md)

---

## Summary

We ran 33 complete Bugmeister counts at two spots in Pankow, about 800 m apart. The Kraft Pit is an unmanaged ruderal patch: grasses, dandelion, clover and plantain on sandy ground, with leaf litter, stones and a fallen branch. The Pankow flower bed is a planted ornamental bed of canna, rudbeckia and creeping jenny on dark bare soil, edged with paving.

- **Total insect counts did not differ clearly.** The Kraft Pit averaged 2.9 insects per minute and the flower bed 2.2 (5-minute counts only). On the 12 days both spots were counted, the Kraft Pit came out higher on 6 days and lower on 6 (paired t = 1.45, df 11, not significant).
- **The mix of insects differed sharply.** The Kraft Pit recorded 10 insect groups and the flower bed 7. A Kraft Pit count found 4.7 groups on average and a flower bed count 2.7. On the 12 paired days, the Kraft Pit found more groups on 9 days and the same number on 3. It never found fewer (sign test p ≈ 0.004).
- **Butterflies, grasshoppers and beetles were never recorded at the flower bed.** At the Kraft Pit, each of these turned up in 8 or 9 of the 20 counts. If butterflies were as common at the flower bed as at the Kraft Pit, the chance of missing them in all 13 counts would be about 0.04 %.
- **Flies dominate the flower bed.** They made up 58 % of its insects, against 33 % at the Kraft Pit. Ants were three times as frequent at the Kraft Pit (3.1 vs 1.0 per 5 minutes). Bees were similar at both spots (0.9 vs 1.2 per 5 minutes).
- **Temperature mattered more than cloud cover.** Counts at the Kraft Pit rose with air temperature (r = 0.63, n = 20) and fell with humidity (r = −0.50). Cloud cover showed no relationship at either spot.

This is a small sample from two observers over five weeks at the end of the season. It shows where to look next but proves nothing about either habitat.

## Approach

### The spots

| | Kiez Café Kraft Pit | Pankow flower bed |
|---|---|---|
| Spot ID / slug | 17 / `kiez-caf-kraft-pit` | 20 / `pankow-flower-bed` |
| Space in the database | Kiez Kaffee Kraft Baustelle | Burgerpark Pankow |
| Coordinates | 52.5700, 13.4085 | 52.5693, 13.3968 |
| Created | 2026-09-06 | 2026-09-07 |
| Character | Spontaneous ruderal patch beside a building site | Planted ornamental bed beside a footpath |
| Plants recorded | Grasses, dandelion, clover, plantain, mugwort, dead-nettle, thistle, aster, daisy | Canna, rudbeckia / black-eyed Susan, helenium, creeping jenny, cinquefoil, hosta, hibiscus, aster, an ornamental vine |
| Ground layer | Dense low herb layer with dry thatch and standing dead stems; patches of sandy soil | Dark bare soil and mulch between plants; leaf litter later in the season |
| Structural extras | Several stones, a fallen branch and twigs, one burrow opening | Stone kerb and paving on the bed edge |
| SPAIA monitor on site | Yes (pole, battery box visible in photos) | Yes (unit on post visible in photos) |

Plant and habitat lists come from the photo taken at the start of each count. Each photo was labelled by DeepSeek vision, and the labels were then corrected by hand (`plant_observations`, `habitat_features`).

### How counts were made

Each count is one Bugmeister session. The observer photographs the spot, watches it for a fixed time, usually 5 minutes, and taps an insect group each time they see one. The groups are Flies, Ants, Bees, Wasps, Butterflies, Grasshoppers, Beetles, Spiders, Bugs and Other. Weather comes from the nearest DWD station via Bright Sky, matched to the start time of the count. Observers can also record wind, notes and other creatures they see.

Two observers made all the counts. Observer A did 11 at the Kraft Pit and 9 at the flower bed; Observer B did 9 and 4. On most paired days one person visited both spots within an hour, so time of day and weather were similar.

### What was included

- 33 completed sessions: 20 at the Kraft Pit and 13 at the flower bed. Two abandoned sessions at the Kraft Pit (0 counts, no completion time) were dropped.
- Rates are insects per minute of the session's set duration. One flower bed count (2026-09-18) was set to 2 minutes, and its figures are shown separately where it matters.
- Times are Berlin local time (CEST).

## Results

### Totals

| | Kraft Pit | Flower bed |
|---|---|---|
| Completed counts | 20 | 13 |
| Minutes watched | 105 | 62 |
| Insects recorded | 287 | 139 |
| Insects per count, mean (median, range) | 14.3 (14, 6–30) | 10.7 (9, 5–23) |
| Insects per minute, 5-min counts only | 2.89 | 2.20 |
| Insect groups recorded (of 10) | 10 | 7 |
| Groups per count, excluding Other | 4.7 | 2.7 |
| Shannon diversity (H′) | 1.92 | 1.31 |
| Evenness (H′ / ln S) | 0.83 | 0.67 |

Counting more often at the Kraft Pit could explain some of its extra groups. To check, we drew 13 of its 20 counts at random, 5,000 times. Every draw still contained at least 9 groups, and the average was 10.0, so the gap is not just extra effort.

### Insect mix

| Group | Kraft Pit total | Share | Per 5 min | Present in | Flower bed total | Share | Per 5 min | Present in |
|---|---|---|---|---|---|---|---|---|
| Flies | 95 | 33 % | 4.75 | 18/20 | 80 | 58 % | 6.62 | 13/13 |
| Ants | 64 | 22 % | 3.05 | 16/20 | 13 | 9 % | 1.00 | 7/13 |
| Other | 35 | 12 % | 1.75 | 17/20 | 25 | 18 % | 1.92 | 9/13 |
| Bees | 20 | 7 % | 0.93 | 11/20 | 11 | 8 % | 1.19 | 6/13 |
| Wasps | 19 | 7 % | 0.90 | 12/20 | 2 | 1 % | 0.15 | 2/13 |
| Grasshoppers | 16 | 6 % | 0.80 | 9/20 | 0 | – | – | 0/13 |
| Butterflies | 15 | 5 % | 0.75 | 9/20 | 0 | – | – | 0/13 |
| Spiders | 11 | 4 % | 0.55 | 8/20 | 6 | 4 % | 0.46 | 5/13 |
| Beetles | 8 | 3 % | 0.38 | 8/20 | 0 | – | – | 0/13 |
| Bugs | 4 | 1 % | 0.20 | 3/20 | 2 | 1 % | 0.15 | 2/13 |

The groups missing from the flower bed all live in or near the ground layer, or need grassy, weedy vegetation: grasshoppers in long grass, beetles in litter, and butterflies, whose larvae feed on wild herbs. The ants fit too, since sandy, undisturbed soil suits nesting. The flower bed's bare dark soil, mulch and paved edge offer little of this. Its flowers draw flies and some bees.

### Same-day comparison

On 12 days both spots were counted. Rates are insects per minute; groups exclude Other.

| Date | Kraft Pit time | Rate | Groups | Flower bed time | Rate | Groups | Weather | °C |
|---|---|---|---|---|---|---|---|---|
| 09-07 | 08:10 | 1.6 | 4 | 08:43 | 1.8 | 4 | overcast | 13.9 |
| 09-10 | 15:44 | 3.4 | 5 | 16:26 | 1.8 | 3 | overcast | 18.6 |
| 09-13 | 15:40, 16:30 | 2.8 | 6.5 | 16:57 | 1.4 | 3 | overcast | 20.3 |
| 09-14 | 08:02, 18:38 | 1.4 | 4.5 | 18:59 | 3.0 | 1 | partly | 15.4 |
| 09-15 | 15:25 | 6.0 | 6 | 14:52 | 4.6 | 2 | overcast | 25.1 |
| 09-18 | 15:39 | 2.8 | 3 | 14:53 | 3.5 * | 2 | sunny | 20.2 |
| 09-19 | 17:05 | 1.6 | 4 | 16:45 | 2.2 | 4 | overcast | 19.5 |
| 09-20 | 12:34 | 1.6 | 5 | 12:04 | 1.8 | 3 | overcast | 19.9 |
| 09-21 | 14:38 | 2.4 | 3 | 16:17 | 1.0 | 1 | overcast | 17.0 |
| 09-26 | 10:21 | 2.8 | 4 | 12:49 | 3.2 | 3 | overcast | 18.5 |
| 09-27 | 12:41 | 4.4 | 4 | 12:13 | 2.4 | 4 | sunny | 20.6 |
| 09-28 | 15:42 | 3.8 | 5 | 18:13 | 1.4 | 3 | overcast | 22.2 |

\* 2-minute count. Weather and temperature are for the flower bed count. Where the Kraft Pit had two counts in a day, the rate and group count are averages.

- Rate: Kraft Pit higher on 6 days, flower bed higher on 6. Mean difference +0.54 insects per minute in the Kraft Pit's favour, not significant.
- Groups: Kraft Pit higher on 9 days, equal on 3, never lower. Mean difference +1.75 groups per count.

### Weather and time of day

| | Kraft Pit | Flower bed |
|---|---|---|
| Rate vs air temperature (r) | 0.63 | 0.39 |
| Rate vs relative humidity (r) | −0.50 | −0.38 |
| Rate vs wind speed (r) | −0.12 | −0.43 |
| Rate vs cloud cover (r) | 0.02 | −0.24 |
| Rate before 12:00 (n) | 1.8 (4) | 1.8 (1) |
| Rate 12:00–15:00 (n) | 3.1 (6) | 2.9 (6) |
| Rate after 15:00 (n) | 3.0 (10) | 1.8 (6) |

The best Kraft Pit counts came on the warmest afternoons: 30 insects at 25.1 °C on 09-15, and 19–22 at 19–24 °C. Counts before 12:00 were mostly low at both spots, at about 1.8 insects per minute. The exception was the sunny, cold Kraft Pit count on 09-26 (11.4 °C, 2.8 per minute). The flower bed fell off in the late afternoon, while the Kraft Pit did not. The bed is the more exposed of the two and may cool faster, but with six counts in each slot this is a lead to follow, not a finding. Wind seemed to matter more at the flower bed, which fits a site of tall, open-stemmed plants. The weather bucket (sunny / partly / overcast) showed no clear pattern. Most counts were overcast, and the station's cloud reading may not match conditions on site.

### Other records

- Other creatures: a snail at the Kraft Pit (09-16) and a mouse at the flower bed (09-26).
- Notes: "Snail trail" at the Kraft Pit (09-20). On 09-26 someone had left a plastic barrier on the Kraft Pit. It covered the plants, killed some dandelions and crushed others. That count still recorded 14 insects, mostly flies and Other.
- The flower bed count on 09-26 logged 10 "Other" out of 16 insects, which is 40 % of all "Other" at that spot. It is worth checking the session photos to see what these were.

### Per-session data

Group abbreviations: Fl flies, An ants, Be bees, Wa wasps, Bu butterflies, Gr grasshoppers, Bt beetles, Sp spiders, Bg bugs, Ot other.

**Kiez Café Kraft Pit**

| Date | Time | Min | Weather | °C | Fl | An | Be | Wa | Bu | Gr | Bt | Sp | Bg | Ot | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 09-06 | 10:54 | 10 | partly | 16.3 | · | 6 | 3 | 2 | · | · | 1 | · | · | · | **12** |
| 09-07 | 08:10 | 5 | overcast | 11.6 | 3 | · | 1 | 2 | · | · | 1 | · | · | 1 | **8** |
| 09-10 | 15:44 | 5 | overcast | 18.6 | 5 | 1 | 1 | · | 4 | 2 | · | · | · | 4 | **17** |
| 09-13 | 15:40 | 5 | overcast | 20.7 | 2 | 5 | · | 1 | · | · | · | 1 | 1 | 2 | **12** |
| 09-13 | 16:30 | 5 | overcast | 20.3 | 1 | 5 | 1 | 3 | 1 | · | 1 | 1 | 1 | 2 | **16** |
| 09-14 | 08:02 | 5 | overcast | 14.1 | 1 | 2 | · | 1 | · | · | 1 | 2 | · | 1 | **8** |
| 09-14 | 18:38 | 5 | partly | 15.4 | 1 | 2 | · | · | 1 | · | 1 | · | · | 1 | **6** |
| 09-15 | 15:25 | 5 | overcast | 25.1 | 10 | 11 | · | · | 1 | 4 | 1 | 1 | · | 2 | **30** |
| 09-16 | 18:12 | 5 | overcast | 18.3 | · | 3 | · | 2 | · | 3 | · | 2 | · | 6 | **16** |
| 09-18 | 15:39 | 5 | sunny | 20.2 | 5 | 4 | · | · | · | 2 | · | · | · | 3 | **14** |
| 09-19 | 17:05 | 5 | overcast | 19.5 | 2 | 4 | 1 | · | · | · | 1 | · | · | · | **8** |
| 09-20 | 12:34 | 5 | overcast | 19.9 | 2 | 2 | 1 | 1 | · | 1 | · | · | · | 1 | **8** |
| 09-21 | 14:38 | 5 | overcast | 17.1 | 7 | 1 | · | · | 3 | · | · | · | · | 1 | **12** |
| 09-22 | 15:39 | 5 | partly | 18.1 | 6 | 2 | 3 | · | 1 | 1 | · | · | · | 1 | **14** |
| 09-23 | 14:28 | 5 | overcast | 19.1 | 11 | · | 3 | 1 | 1 | 1 | · | 1 | 2 | 2 | **22** |
| 09-26 | 10:21 | 5 | sunny | 11.4 | 5 | · | · | 2 | 2 | · | 1 | · | · | 4 | **14** |
| 09-27 | 12:41 | 5 | sunny | 20.6 | 11 | 7 | 1 | · | · | · | · | 1 | · | 2 | **22** |
| 09-28 | 15:42 | 5 | overcast | 23.8 | 8 | 6 | · | 1 | · | 1 | · | 2 | · | 1 | **19** |
| 09-29 | 14:55 | 5 | partly | 24.5 | 7 | 3 | 4 | 1 | 1 | · | · | · | · | 1 | **17** |
| 10-14 | 13:56 | 5 | overcast | 15.3 | 8 | · | 1 | 2 | · | 1 | · | · | · | · | **12** |

**Pankow flower bed**

| Date | Time | Min | Weather | °C | Fl | An | Be | Wa | Bu | Gr | Bt | Sp | Bg | Ot | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 09-07 | 08:43 | 5 | overcast | 13.9 | 3 | · | 1 | 1 | · | · | · | 1 | · | 3 | **9** |
| 09-10 | 16:26 | 5 | overcast | 18.6 | 4 | 3 | 2 | · | · | · | · | · | · | · | **9** |
| 09-12 | 14:10 | 5 | sunny | 18.9 | 7 | · | · | · | · | · | · | · | 1 | 1 | **9** |
| 09-13 | 16:57 | 5 | overcast | 20.3 | 3 | 1 | · | · | · | · | · | · | 1 | 2 | **7** |
| 09-14 | 18:59 | 5 | partly | 15.4 | 12 | · | · | · | · | · | · | · | · | 3 | **15** |
| 09-15 | 14:52 | 5 | overcast | 25.1 | 18 | 2 | · | · | · | · | · | · | · | 3 | **23** |
| 09-18 | 14:53 | 2 | sunny | 20.2 | 4 | · | 3 | · | · | · | · | · | · | · | **7** |
| 09-19 | 16:45 | 5 | overcast | 19.5 | 5 | 1 | 3 | 1 | · | · | · | · | · | 1 | **11** |
| 09-20 | 12:04 | 5 | overcast | 19.9 | 5 | 2 | · | · | · | · | · | 1 | · | 1 | **9** |
| 09-21 | 16:17 | 5 | overcast | 17.0 | 4 | · | · | · | · | · | · | · | · | 1 | **5** |
| 09-26 | 12:49 | 5 | overcast | 18.5 | 3 | 1 | · | · | · | · | · | 2 | · | 10 | **16** |
| 09-27 | 12:13 | 5 | sunny | 20.6 | 7 | 3 | 1 | · | · | · | · | 1 | · | · | **12** |
| 09-28 | 18:13 | 5 | overcast | 22.2 | 5 | · | 1 | · | · | · | · | 1 | · | · | **7** |

## What we learned

1. **Counting groups tells the two spots apart; counting individuals does not.** Total counts depend on weather and hour, and they swamp any habitat effect over a dozen paired days. The number of groups per count separated the spots on almost every day. For short citizen counts like these, groups per count is the better headline number.
2. **The unmanaged patch is the richer habitat.** In this data, the weedy patch with litter, stones and bare sandy soil supports ground-layer insects that the tidy ornamental bed lacks. That matches the habitat features the photo labels picked up, and it gives a concrete example for the habitat score.
3. **Paired visits work.** Visiting both spots in one trip kept weather and season the same between them. The best pairs were within an hour of each other. Wider gaps, such as 10:21 vs 12:49 on 09-26 or 15:42 vs 18:13 on 09-28, mix in time-of-day effects.
4. **"Other" is too big to ignore.** It was 12–18 % of everything recorded. Photos or a short follow-up prompt would make it usable.

## Data quality

- **Wrong space for the flower bed (fixed 2026-11-23).** Spot 20 and its 13 sessions were under "Volkspark Humboldthain, Wedding" (space 2), although the coordinates were correct. They have been moved to "Burgerpark Pankow" (space 9). The Kraft Pit's space still has no locality set.
- **Uneven durations.** One flower bed count was set to 2 minutes (09-18). The first Kraft Pit count was 10 minutes. One flower bed count (09-21) is stored as 5 minutes but finished after 3.6 minutes of wall-clock time.
- **Abandoned sessions.** Two Kraft Pit sessions (09-29 08:12, and an anonymous one on 10-14 11:55) have no completion time and no sightings. One has weather attached.
- **Optional fields left empty.** The mood fields are empty in every session. Observed wind was filled in for 9 of 33 counts.
- **No device data yet.** Both spots have a SPAIA monitor in the photos, but `device_readings` has no rows for either spot. A comparison of hand counts and device counts is not possible yet.
- **Habitat-eval images are unlabelled.** There are 21 eval items for the Kraft Pit and 13 for the flower bed, with 0 labels. Labelling them would provide a time series of habitat change alongside these counts.
- **Weather is not measured on site.** Weather comes from the nearest DWD station, not the spot. The "sunny" bucket here means 0 % station cloud cover.

## Open questions

- TODO: Do the device monitors at these spots log anything, and if so, where is it going?
- TODO: What were the 10 "Other" at the flower bed on 09-26? Check the session photo and ask the observer.
- Do butterflies, grasshoppers and beetles really never reach the flower bed, or were they missed? A few longer counts (10–15 minutes) or a summer revisit would answer this.
- Was the flower bed's late-afternoon drop caused by shade, cooling or chance? Matched-time counts would tell.
- How did the 09-26 plastic barrier affect the Kraft Pit's plants and insects in the counts that followed? The habitat-eval image pairs could show this once labelled.
- Does the difference hold in spring and summer, when the flower bed is in full bloom? This data covers only September and early October.

## Sources

Queries were run against the remote D1 database with `wrangler d1 execute bugmeister --remote`. Spot IDs 17 and 20. All session, sighting, weather, creature, plant and habitat rows for those spots were included. Abandoned sessions are excluded where noted. Statistics (means, Pearson r, paired t, sign test, rarefaction by random subsampling) were computed in Python from the exported rows. Observer IDs are anonymised as A and B.
