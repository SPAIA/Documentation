# SPAIA dashboard (Angular web app)

**Status:** Prototype, last commit November 2020. Live camera view works. Tagging page shows photos and boxes but does not save edits. Draft built from the repo, so items marked TODO need confirming with whoever ran it.
**Period:** February to November 2020, 14 commits by one author
**Source:** Private repo `playstate_africa/spaia-dashboard` on Bitbucket
**Related:** The ESP32-CAM work that fed it is in [early_camera_experiments.md](../hardware_experiments/early_camera_experiments.md). The REST API that came after it is in [api_infrastructure.md](api_infrastructure.md), with its tables in [database_structures.md](database_structures.md).

---

## Summary

- What it is. A browser app with two jobs. Watch what an ESP32 camera sees, live, and review captured insect photos with boxes drawn on them.
- Why it existed. In 2020 the work was detecting insects from the camera image alone. A live view of the raw data and a way to judge what was caught fit that. The repo has no notes on intent, so this is my reading. TODO confirm.
- Result in one line. The live view works. The tagging page was a start. Its data model was too thin, and the 2021 REST API replaced it.

## Stack

| Part | Choice |
|---|---|
| Framework | Angular 9.0 with TypeScript 3.7 |
| UI | Angular Material 9 and Angular Flex Layout |
| Login | Auth0 SPA SDK, tenant `spaia-dev` in the EU region |
| Live data | MQTT over secure WebSocket through `ngx-mqtt`, broker `mqtt.spaia.co.za` on port 8083 |
| Stored data | Cloud Firestore through AngularFire |
| Images | DigitalOcean Spaces, public CDN path `uploads/` |
| Hosting | Firebase Hosting, project `spaia-9d111`, single-page rewrite to `index.html` |

The Firebase project is the same one the REST API later used to verify user tokens.

## Pages

All three routes sit behind an `AuthGuard`, which sends signed-out users to Auth0 and returns them to the page they asked for.

| Route | Component | What it does |
|---|---|---|
| `/debug/camera` | `CameraComponent` | Subscribes to the MQTT topic `camera/raw` and paints each message on a canvas as a grid of grey squares |
| `/tag` | `TagComponent` | Loads the Firestore `events` collection, shows one photo at a time with its boxes, and steps through them |
| `/profile` | `ProfileComponent` | Shows the Auth0 profile |

### Live camera view

- Each MQTT message is a flat byte array. The component reads 1,200 values as 30 rows of 40 columns and draws each as a grey cell, sized to fit the window.
- It redraws on every message and on window resize. There is no frame rate limit and no buffering.
- A leftover loop in `ngOnInit` groups a 160 by 120 frame, 19,200 pixels, into 192 cells of 10 by 10 pixels. That matches the per-cell averages the ESP32 firmware computed. It only builds an array that nothing reads.
- TODO. Which firmware version published these frames, and whether the 40 by 30 size is the camera frame downsampled or the cell grid. The two numbers do not line up.

Why it matters. Averaging inside a cell hides a small insect, which is why the 2020 approach was dropped. See the 2020 write-up for the reasoning.

### Tagging page

- Photos come from Firestore, kept in sync live. A photo added by a device shows up without a refresh.
- Each Firestore document has `url`, the image path under the Spaces bucket, and `rectangles`, an array of boxes. The code reads each box as `x`, `y`, `width`, `height` and `title`.
- The page loads the image, measures its true width and works out a scale factor against the displayed width, so the boxes land in the right place at any window size.
- Clicking a box opens a Material dialog with a text field for the insect name. The answer goes into the box's `title`.
- Next and previous buttons wrap around. Delete removes the Firestore document for the current photo.

Things that were left unfinished.

- Editing a box title or deleting a box changes only the copy in the browser. Nothing writes back to Firestore. Only deleting a whole photo is saved.
- The species list in `BoundingBoxComponent` is a placeholder with two names, `bob` and `bill`.
- There is no way to draw a new box. The drag handlers only log to the console.
- No pagination. It loads the entire collection on page load.
- No record of who tagged or verified anything. Firestore documents have no author fields.

## Data flow

```
ESP32-CAM --MQTT/WSS--> mqtt.spaia.co.za --> /debug/camera  (live grid)

device/model --> Spaces upload + Firestore `events` doc --> /tag  (review, box, name)
```

The second path has no uploader in this repo. Whatever wrote the Firestore documents and the Spaces files lives somewhere else. TODO find it, probably the Pi detector or a script.

## What the 2021 API changed

| Dashboard in 2020 | REST API in 2021 |
|---|---|
| Auth0 login | Firebase Authentication |
| Firestore document per photo | Relational tables for events, media, regions, labels and sensor readings |
| Boxes and names inside the photo document | Separate Region and Label rows, many labels per box |
| Browser talks to the database directly | Browser talks to an API that checks who is calling |
| MQTT for live frames | No live view. Devices batch-upload a CSV and images |
| Images on DigitalOcean Spaces | Images on Cloudflare Images |

The change in the second row matters most. In Firestore, a human's tag is a text field in the photo. In the API, it is a row that says which region, which label and which user verified the event, so tags can be counted, corrected and used as training data.

## Issues found in the repo

- The MQTT username and password are written into `app.module.ts`, so they ship to every browser. Treat the broker as publicly reachable and rotate the credentials if the broker still exists.
- The Firebase web config and the Auth0 client ID are in `environment.ts` and `auth.service.ts`. Both are meant to be public. Real protection depends on Firestore security rules and the Auth0 allowed URLs, which are not in the repo. TODO check them.
- `rejectUnauthorized: false` on the MQTT client turns off TLS certificate checking.
- Angular 9 and Firebase 8 are end of life. Reviving it means a framework upgrade, not a patch.
- Component spec files are Angular CLI stubs. There are no real tests.

## What to carry forward

Keep.
- A live view of what the sensor sees. It is the fastest way to judge a new detector, and the event camera work will want one.
- A review screen with a box on a photo and a name field. It is the right shape for labelling data.
- Sign-in before anything else loads.

Change.
- Talk to the REST API for tags, so they persist and have an author.
- Replace the placeholder species list with a search against the `Labels` table.
- Add box drawing and resizing.
- Show the sensor readings from the event beside the photo.

## Open questions

- What wrote to Firestore and Spaces? TODO
- Was the dashboard still used after the REST API arrived in July 2021, or was it dropped? TODO
- Is the broker at `mqtt.spaia.co.za` still running? TODO
- The newer client for the REST API is [reviewer_dashboard.md](reviewer_dashboard.md). Was this one still in use once that existed? TODO
