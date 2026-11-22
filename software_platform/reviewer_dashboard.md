# SPAIA reviewer dashboard (dashboard-2)

**Status:** Prototype, last commit 17 August 2021. It is the client for the REST API. Sign-in, device and event lists, and a box-drawing screen work against a local server. Labelling was started on the last day and not finished. Draft built from the repo, so items marked TODO need confirming with whoever ran it.
**Period:** 12 July to 17 August 2021, 24 commits
**Source:** Private repo `playstate_africa/dashboard-2` on Bitbucket
**Related:** The server it calls is in [api_infrastructure.md](api_infrastructure.md) and its data in [database_structures.md](database_structures.md). The 2020 version it replaced is in [dashboard_and_tagging.md](dashboard_and_tagging.md).

---

## Summary

- What it is. An Angular 12 web app for people who review what the field devices capture. It lists devices, shows the photos from the ones you pick, lets you draw and resize a box around each insect, and shows the temperature and humidity logged with the photo.
- Why it exists. It replaces the 2020 tagging page, which kept boxes as text inside a Firestore document. This one works on the Regions, Labels and SensorData tables of the REST API.
- Result in one line. The right screen, built in five weeks. It was never pointed at a live server, and the data it expects differs from what the final API returns, so it could not have shown a real photo as it stands.

## Stack

| Part | Choice |
|---|---|
| Framework | Angular 12.1, TypeScript 4.3, Angular Material and Flex Layout |
| Login | Firebase Authentication through AngularFire, Google sign-in popup |
| API client | Typed services and models, generated from the OpenAPI spec. The style matches `ng-openapi-gen`, which is not in `package.json`, so the generating step is not reproducible from the repo |
| Hosting | Firebase Hosting, target `spaia-dashboard` in project `spaia-9d111`, the same target the 2020 dashboard used |
| Tooling | Husky pre-commit running lint-staged and Prettier |
| Tests | Nine Angular CLI spec stubs. No real tests |

The Firebase project is shared by three things. The 2020 dashboard, this one, and the REST API that verifies the tokens.

## Screens

| Route | Guard | What it does |
|---|---|---|
| `/` | none | Splash screen. A set of test buttons that call the API, described below |
| `/classification` | signed in | The review screen |
| `/profile` | signed in | Calls `GET /me` and logs the result. Shows nothing |

The side navigation also links to `/debug/camera`, `/devices` and `/tag`. None of these routes exist in this repo, so they are left over from the 2020 app or planned.

### Sign-in

- Login is a Google popup, started from the account menu in the nav bar.
- An HTTP interceptor adds `Authorization: Bearer <Firebase ID token>` to every request made through Angular's HTTP client. It does not check the host, so the token would go to any URL the app called.
- `AppComponent` also writes the token to `localStorage`. Nothing reads it back.
- Protected routes use `AngularFireAuthGuard`. There is no sign-in page, so a signed-out visitor to `/classification` gets no prompt. They have to find the menu first.

### Classification screen

The page has three parts.

1. Image area. The selected photo with one box per Region, placed by dividing the stored pixel values by a scale factor worked out from the photo's real width against its displayed width. Each box can be dragged. A thin handle on the right edge changes its width, and one on the bottom edge changes its height. Each box shows its first label and a delete icon.
2. Sidebar with three tabs.
   - Filters. A device checklist built from `GET /devices`. Ticking devices calls `GET /events` for those IDs. The Date range panel is empty.
   - Labels. The labels on the selected box, and a list of saved labels from `GET /labels`. The remove button only logs to the console.
   - Data. The temperature, and every reading on the event, with the sensor type name.
3. Filmstrip. A strip of the event photos along the bottom. Clicking one makes it the active event. Events with no image show a bug icon.

### Splash screen test buttons

These exist to exercise the API by hand.

- Generate random event. Posts five demo events for device 1, all dated now, with a point near Johannesburg and made-up image names such as `test 4`.
- View devices. Lists the caller's devices.
- Link a device. Registers the name `demo-ResearchAssistant` to the signed-in user.
- Get events. Fetches events for devices 1 and 2.
- Add a device. Does nothing.

## What works and what is unfinished

| Feature | State |
|---|---|
| Google sign-in, token sent to API | Works |
| Device list and event list | Works against the API client as generated |
| Resize a box, save to the API | Saves when the mouse is released on a handle |
| Move a box | Moves on screen and updates the local object. It reaches the API only if a resize follows, because the resize sends the whole box |
| Delete a box | Removes it from the screen only. `DELETE /region/{id}` is not called |
| Add a box | Not built |
| Add or remove a label on a box | Not built. The last commit, "labels started", only adds the lists |
| Date range filter | Empty panel |
| Verify an event | Not built, and not in the client |
| Paging | Not built. The call asks for the first page and the user cannot move on |

## Why it could not have worked against the final API

The client was generated from an early spec. By the time the API was finished the two had drifted.

| Client expects | API returns or accepts | Effect |
|---|---|---|
| `event.image`, a URL string | An `EventMedia` list under `media`, with a Cloudflare image ID | The image area shows the broken-image icon, and the filmstrip shows bug icons |
| `sensorData[].Sensor.SensorType.name` | Sensor data rows with no nested sensor | The temperature field stays blank and readings show no names |
| Update and delete an event by `deviceId` in the path | `eventId` | Generated calls for these two operations use the wrong path parameter |
| Events query with `deviceId`, `before`, `limit` | Also `after` and `offset` | Cannot page or filter by start date |
| Label list `offset`, `limit` | Also `search` and `eventIds` | Cannot search labels |
| No call for creating regions or labels, linking labels, verifying events, settings or linking a device | These endpoints exist | Not reachable from the app |

There is a second reason. `ApiConfiguration` sets the root URL to `http://localhost:8080`, and nothing overrides it. The dev server URL, `https://api.spaia.co.za`, appears only in the API spec. A deployed build would call the visitor's own machine.

## Other issues found in the repo

- `environment.prod.ts` has no Firebase config. With the standard Angular file replacement, a production build starts Firebase with no config, and sign-in would fail. Only `environment.ts` carries it. TODO confirm against `angular.json` and a real build.
- The Firebase web config is committed in `environment.ts`. That is normal for Firebase, since the keys identify the project and do not grant access. Real protection is the API's token check and Firebase's allowed domains, not in this repo. TODO check.
- 13 `console.log` calls are left in, including one that logs the full Google sign-in result.
- A window-resize handler is attached to the wrong property. The `@HostListener` decorator sits on `dragPosition`, so resizing the window does not recalculate the box scale.
- No error handling on any API call and no loading states.

## What to carry forward

Keep.
- The layout. Image in the middle, device and label controls on the side, strip of photos along the bottom, is what a reviewer wants.
- Box and handle editing in pixel coordinates. It maps directly to the Region table.
- Putting sensor readings next to the photo. It gives context for the call.
- A typed client generated from the API spec, once the generating step is in `package.json`.

Change.
- Regenerate the client from the final `openapi.yaml`, and add a script to do it.
- Read images from `media` and build URLs from the Cloudflare image ID.
- Save moves, deletes and new boxes to the API. Add labels to boxes with `POST /label/region/`.
- Add a verify button, wired to `PUT /event/{id}/verify`, so a reviewed event is stamped with who checked it.
- Fix the production environment and the API root URL, and send the token only to the API host.
- Add a sign-in page and paging.

## Where it sits in the story

| Date | Event |
|---|---|
| Feb to Nov 2020 | First dashboard on Firestore and MQTT |
| 24 Jul 2021 | REST API repo begins |
| 12 Jul to 17 Aug 2021 | This client is built beside it, with the API still being written |
| Aug 2021 to Jun 2022 | API continues. This client does not |

The two repos were built at the same time. The client's last commit is the same day the API gained its label list. The API then added settings, image upload and CSV import, which this client never uses. TODO ask whether a later client exists for that work.

## Open questions

- Was this ever used with real photos, or only with the demo events from the splash screen? TODO
- Is there a later version in another repo, built for the Cloudflare image IDs and the verify endpoint? TODO
- Was it ever deployed to Firebase Hosting, given the broken production environment? TODO
- Should reviewing stay a separate web app, or move into whatever the next camera rig produces?
