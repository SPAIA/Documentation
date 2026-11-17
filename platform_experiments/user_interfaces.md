# User interface experiments (2020 to 2026)

**Status:** Ongoing. Five generations of interface, from a marketing site and an Angular dashboard to a mobile-first citizen science app. Draft built from the repos, so usage, feedback and the reasons for each change are marked TODO unless the code says why.
**Period:** February 2020 to October 2026
**Source:** GitHub `SPAIA/spaia-dash`, `pledge`, `device_api`, `lab`, `app` and Bitbucket `spaia_web`, `spaia-dashboard`, `dashboard-2` (all private)
**Related:** [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md), [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md)

---

## Summary

- Audiences. Three groups use the interfaces. Device owners who need to set up and watch a device. People who label what a device saw. Members of the public who want to count insects or protect a patch of land.
- Path. The first interfaces were for staff and device owners. Since 2024 the work has shifted to the public, with a device commissioning wizard, a field survey form, a land pledge map and a gamified counting app.
- Stack. Angular with Material gave way to SvelteKit in 2024. The component library changed three times (Material, Flowbite, DaisyUI) and settled on shadcn-svelte in September 2026.
- Results. TODO. No usage numbers or user feedback are in the repos.

## Generations compared

| Generation | Period | Stack | Audience | Main screens |
|---|---|---|---|---|
| Marketing site | Apr 2020 to Jan 2021 | Hugo static site, Cloudflare Workers Sites | Public | Landing and content pages |
| Angular dashboard 1 | Feb 2020 to Jul 2021 | Angular 9, Firebase hosting, Auth0, MQTT | Staff and device owners | Live device data |
| Angular dashboards 2 and 3 | Jul 2021 to May 2024 | Angular Material, Mapbox, Firebase auth | Staff, device owners, labellers | Device list and settings, classification, data explorer |
| Svelte web apps | Feb 2024 to Jun 2025 | SvelteKit on Cloudflare, Flowbite, Mapbox, Supabase | Public and device owners | Habitat pledge, device wizard, transect survey, simulators |
| Bugmeister | Aug 2026 onwards | SvelteKit on Workers, Tailwind 4, shadcn-svelte, MapLibre | Public | Timed insect counts, leaderboard, spots and spaces, habitat scan |

## Marketing site (`spaia_web`)

- A Hugo site with a custom `spaia` theme and an English content folder set up for more languages. It began with a Netlify config on 28 Apr 2020. By the end it built through Bitbucket Pipelines, which installed Hugo and Wrangler and published to Cloudflare Workers Sites.
- 130 commits between Apr 2020 and Jan 2021, with a `develop` branch.
- Images came from an image CDN hostname set in the site config.
- TODO is the site still live, and was it replaced.

## Angular dashboards

**`spaia-dashboard` (Feb 2020 to Jul 2021, 49 commits).**

- Generated with Angular CLI 9. Hosted on Firebase, with Auth0 login and live data over MQTT (`ngx-mqtt`).
- Two experimental branches. `apollo` (Nov 2020) tried a GraphQL data layer. `api-based` (Jul 2021) switched to the REST API.

**`dashboard-2` (Jul to Aug 2021, 24 commits).**

- A rebuild with Angular Material and flex layout, with an API client folder for the new Node API.
- The last commit, on 17 Aug 2021, is "labels started". The classification screen begins here.

**`spaia-dash` (GitHub, Jul 2021 to May 2024, 187 commits).** The longest-running interface.

- **Devices.** A device list, create and add dialogs, device settings pages (camera, field agent firmware, system) and a locator that places a device on a Mapbox map (Sep 2021).
- **Classification.** A screen for reviewing events, with a filmstrip of images, a sidebar, region drawing and a label dialog. Covered in [data_context_validation_and_augmentation.md](data_context_validation_and_augmentation.md).
- **Data explorer.** Filters for events with or without activity and with or without labels, a sidebar and charts (Apr 2024). Branches `charts`, `data-filter`, `env` and `dev` carry this work.
- **Accounts.** Register, login, forgot password and email verification, using Firebase.
- **OpenCV branch.** Mar 2022. Browser-side OpenCV for region guessing.
- Last commit 18 May 2024 ("dev setup"), after which the Svelte apps take over.

## Svelte web apps (2024 to 2025)

**`pledge` (Feb to Nov 2024).** A public page for land owners.

- A visitor signs up, picks a space type, draws a habitat patch on a map (Mapbox GL Draw with geocoder, area measured with Turf), selects pledges, and goes through payment options.
- Data model. Habitats with a GeoJSON feature and area, pledges, habitat types and the links between them.
- Branches for payments, profile, map animation and geocoding. TODO whether it was launched, and the pledge text.

**`device_api/svelte-dashboard`.** A Flowbite Svelte dashboard with Firebase login (Jan 2025), for device owners.

**`lab` (Dec 2024 to Jun 2025, 40 commits).** "A citizen science page for SPAIA devices" (repo description). It brings several experiments together.

- **Device commissioning wizard.** Five steps to add a device. Name it, fill in Wi-Fi details, choose how often it uploads (once daily, hourly, all the time, or only on restart) and plug it into a computer. The Wi-Fi and schedule settings are saved as a `config.json` the person downloads and copies to the device. The Wi-Fi password is encrypted in the browser first (AES-CBC), and the firmware in [xiao_sense_esp32s3_2024.md](../hardware_experiments/xiao_sense_esp32s3_2024.md) decrypts it. The key is shared by both, so this is obfuscation more than security. A separate page redirects firmware downloads to a storage bucket.
- **Device pages.** A "my lab" overview, device cards, device detail pages and a locations page for a pilot site.
- **Field observation form.** A step-by-step transect survey with instructions, weather, season, wind, a consent box and one sighting slide per insect group, optionally with a photo (May 2025). It posts to the API as a field observation.
- **Maps.** Mapbox with drawing and a geocoder (May to Jun 2025).
- **Browser emulator.** A page that runs motion detection on a webcam in JavaScript, at 320x240 with a difference threshold of 7, an adjustable trail length and a Kalman filter per track. It lets someone see how the device's detector behaves without a device. See [event_camera_tests.md](../hardware_experiments/event_camera_tests.md) for the tracking approach it echoes.
- **Bug zoo.** A simulation of insects that appear at a set rate per hour, with speed sliders. It looks like a way to show what a day of detections would look like. TODO confirm its purpose. Branch `bugzoo` holds the first version (Dec 2024).
- **Accounts.** Supabase login with password reset (Dec 2024 to Feb 2025). The Flowbite library was removed once ("bye bye flowbite", Dec 2024), then used again.

## Bugmeister (`app`, 2026)

Spec title: "SPAIA x Kiezwald, gamified citizen science insect observation for Berlin". Built in six weeks.

- **Core loop.** An anonymous visitor picks a spot, chooses a timer of 1, 3, 5 or 10 minutes (10 is the default), taps a tile for each insect group they see, and gets a result card to share. Signing up is asked for after the first session, and the anonymous session is merged into the account.
- **Leaderboard first.** The spec places the district leaderboard before the counting screen on purpose, so people see the stakes before putting in the effort. TODO whether this was tested.
- **Locations.** Hubs and QR posters, so a poster at a park opens the right spot. Later versions rename hubs to spots and spaces, add drawing a space boundary, and add a habitat scan page that gives a photo a score (three free scans per visitor, then sign-up).
- **Languages.** English, German and Dutch, with German route names for some pages. Insect names and level titles are looked up by key.
- **Components.** The spec chose DaisyUI. The code migrated to shadcn-svelte on 13 Sep 2026, then restored the rounded corners it had before.
- **Offline use.** Sessions are saved as a whole snapshot, so a lost connection in a park doesn't lose a count. See [edge_hosting_and_database_architectures.md](edge_hosting_and_database_architectures.md).
- **Simplification over time.** Migrations drop the rarity (0004) and XP (0006) columns, so the points and rarity system in the spec was removed within weeks. TODO why.
- **Devices.** Camera rigs post readings to a spot, so a spot can show human counts next to a device's count.
- **Admin.** Data export, backfilling weather and promo and partner codes that unlock free spaces or spots without payment, with a cap on uses and an audit of each redemption.

## What changed between generations

| Question | Early answer | Later answer |
|---|---|---|
| Who is the user | Staff and device owners | The public |
| Framework | Angular 9 and Material | SvelteKit |
| Maps | Mapbox GL | Mapbox GL, then MapLibre with Terra Draw |
| Auth | Auth0, then Firebase | Supabase, then better-auth |
| Layout | Desktop dashboards | Mobile first, offline tolerant |
| Language | English | English, German and Dutch |

## Open questions

- Usage and feedback. Number of users per interface, completion of the five-step device wizard, drop-off in the field survey, and whether the leaderboard-first order changed behaviour.
- Which interfaces are live today, and which should be retired.
- Whether the browser emulator and Bug zoo were used for demos, tests or both.
- Accessibility and low-bandwidth testing, which the repos don't mention.
