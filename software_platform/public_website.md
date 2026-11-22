# SPAIA public website (spaia_web)

**Status:** Last commit August 2020. A one-page landing site with a sign-up form. It is not the app people use to review insect data. Draft built from the repo, so items marked TODO need confirming with whoever ran it.
**Period:** April to August 2020, 127 commits
**Source:** Private repo `playstate_africa/spaia_web` on Bitbucket
**Related:** The data side is in [api_infrastructure.md](api_infrastructure.md), [database_structures.md](database_structures.md) and [dashboard_and_tagging.md](dashboard_and_tagging.md).

---

## Summary

- What it is. A static site built with Hugo and a custom theme, served at `spaia.co.za` from Cloudflare.
- What it does. Tells visitors what SPAIA is, shows a diagram of the SPAIA-I insect monitor, and collects email addresses for updates.
- What it does not do. It has no login, makes no calls to the REST API, and reads no device data. A search of the repo for the API host, Firebase, MQTT and HTTP client calls finds nothing.
- Result in one line. A working public page. The client for the REST API is in [reviewer_dashboard.md](reviewer_dashboard.md).

## What a visitor sees

The home page has two full-height sections, hand-written in `themes/spaia/layouts/index.html`.

1. Hero. The SPAIA logo, a New York Times quote about an insect world quietly going missing, and a "Sign up for updates" button that opens a MailerLite form.
2. Product. "Fig 1. SPAIA-I Insect Monitor", with an illustration of the device and a bug. The text describes a low-cost internet-connected device that monitors insects and the microclimate around them, and feeds a globally accessible database. The name is pronounced "spyer".

The footer carries a COVID-19 link to `sacoronavirus.co.za`, left over from spring 2020. The navigation is commented out, so there is only one page to see.

The pitch on the page matches what the later repos built. The microclimate readings are the temperature and humidity rows in [database_structures.md](database_structures.md), and the "globally accessible database" is the MariaDB behind the REST API.

## Stack

| Part | Choice |
|---|---|
| Generator | Hugo extended. The Dockerfile pins 0.55.0, the Forestry preview 0.69.0, and the pipeline builds Hugo from source at whatever the current head is |
| Theme | Custom, `themes/spaia`, on Bulma 0.8 with Sass, PostCSS and autoprefixer |
| Hosting | Cloudflare Workers Sites, deployed with Wrangler 1. Worker `spaia-marketing`, route `spaia.co.za/*` |
| Build and deploy | Bitbucket Pipelines on every push to `master` |
| Editing | Forestry.io and Netlify CMS configs, both present |
| Images | imgix, through a Hugo shortcode and the `imgixURL` setting |
| Email sign-up | MailerLite universal script and a pop-up web form |
| Analytics | Google Analytics, Universal Analytics type |

## Build and deploy

On a push to `master`, the pipeline does the following in an `avcosystems/golang-node` image.

1. Clones the Hugo repo and builds the extended version with `go install`.
2. Installs `postcss-cli`, `autoprefixer` and Wrangler globally, then the theme's npm packages.
3. Runs `hugo`, which writes the site to `public/`.
4. Runs `wrangler publish --release`, which uploads `public/` to the Cloudflare Worker.

The Cloudflare key and email come from pipeline variables, not from the file. I checked the full history and found no literal key. `wrangler.toml` holds the account and zone IDs, which identify the account but do not grant access.

## Content model

- Two content types are set up. Pages, and Projects with a title, date, description, author, tags, featured image and background image. The project layouts show cards with imgix-cropped images.
- The repo holds almost no content. `content/en/` has an About page and `content/test.md`, and both are drafts. No project pages are committed, so the Projects layouts render nothing.
- The repo has 44 image files outside `node_modules`, mostly in `static/uploads/` and the theme.

## Where it came from

The repo started as a copy of the Playstate agency site, not as a SPAIA site. Signs of that.

- The About text and the page description say "We build cool stuff with startups".
- The theme script holds a word list of "startups, 3D printers, insurers, bees, Arduinos, Wordpress" and others, written for rotating text. The SPAIA home page has no element for it, so it never shows, but it is agency copy.
- `static/uploads/` holds images from other projects, with names such as the-bloc, abacus and beeadmindash.
- The Netlify CMS config points at `playstate_africa/playstate.git`, and `.gitignore` names `themes/playstate`. Neither is this repo.
- The Forestry config reads media from `media.playstateprojects.com`.

The result is a site that deploys cleanly but still carries another company's defaults under the SPAIA home page.

## Issues found in the repo

- `node_modules/` is committed, 1,029 files and 5.7 MB. Bulma's `node_modules` under the theme is committed as well, because the `.gitignore` entry names the wrong theme.
- `.git` is 25 MB, mostly from images in `static/uploads/`.
- The Dockerfile is named `Dokerfile` and the pipeline does not use it.
- The pipeline builds Hugo from the latest source on every run, so a Hugo release can break the site with no change to the repo.
- Wrangler 1 and Workers Sites are legacy tools. A redeploy today needs a move to current Wrangler and Workers static assets, or to Cloudflare Pages.
- The Universal Analytics property stopped collecting data in July 2023. The site has no working analytics unless it was moved to Google Analytics 4. TODO check.
- The site description and the home page alt text have spelling errors, "Apocolypse" and "AGANIST", and "Biodeversity" in the language settings.
- Two CMS configs, Forestry and Netlify CMS, and neither is wired to this repo's content.
- The page title is just "SPAIA" and the meta description reads "We build cool stuff with startups", which is the agency line.

## What to carry forward

Keep.
- The one-page structure. A clear pitch, one diagram and one call to action is enough for an early project.
- Static hosting. There is nothing here that needs a server.
- A mailing list sign-up. It is the only way the site records interest.

Change.
- Strip the agency copy, uploads and CMS configs, and remove `node_modules` from git.
- Pin the Hugo version and use one CMS, or none.
- Replace the 2020 monitor illustration with a photo or render of the current camera rig, and update the text to match what the device now does.
- Move analytics to a current tool.
- If a public map or live counts are wanted, read them from the REST API's events endpoint. That needs an unauthenticated read-only route, which does not exist yet.

## Open questions

- Is `spaia.co.za` still serving this build? TODO
- The reviewer interface is a separate repo, see [reviewer_dashboard.md](reviewer_dashboard.md). Should the public site ever link to it or show live counts?
- Who owns the MailerLite list and has it been used since 2020? TODO
- Should the site describe the current camera and event camera work, or stay a general project page?
