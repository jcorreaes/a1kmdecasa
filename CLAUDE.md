# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this project is

**a1kmdecasa** ("a 1 km de casa" — "1 km from home") is a small single-page web
app that renders an interactive map and draws a **1 km radius circle** around a
location. It was built around Spain's COVID-19 lockdown rule that limited
outdoor activity (e.g. running) to within 1 km of home, answering the question
"¿Hasta dónde puedo llegar en 1km?" ("How far can I go in 1 km?").

- Live deployment: https://a1kmdecasa.now.sh/
- UI language: Spanish. Default map view is centered on Madrid.

## Tech stack

- **Vue 3** (`vue@^3.4`) — Options API, single root component.
- **Vue CLI 5** (`@vue/cli-service`) — dev server, build, lint (webpack under the hood).
- **Leaflet** (`leaflet@^1.9`) via **`@vue-leaflet/vue-leaflet`** — map rendering.
- **Pug** for SFC `<template>` blocks, **SCSS** for `<style>` blocks.
- **Babel** (`@vue/cli-plugin-babel/preset`) + **core-js** for transpilation.
- **ESLint** (`plugin:vue/vue3-essential` + `eslint:recommended`).

There is **no TypeScript, no test suite, and no router/store** — it is intentionally minimal.

## Commands

```bash
npm install        # install dependencies
npm run serve      # dev server with hot reload (http://localhost:8080)
npm run build      # production build -> ./dist (gitignored)
npm run lint       # eslint with --fix on src
```

Always run `npm run lint` after changing source; CI/lint config is the only
automated check in this repo. There are no unit/e2e tests to run.

## Project structure

```
.
├── public/
│   ├── index.html                # HTML shell: meta tags, OG/Twitter cards, Google Analytics (gtag UA-9706914-3)
│   ├── favicon.ico
│   ├── a1kmdecasa.png            # social share image
│   ├── icons8-marker-40.png      # home marker icon (served as static asset)
│   └── data/geojson/
│       └── Calles_Peatonales_20200513.geojson   # ~2MB GeoJSON of pedestrian streets (Madrid), fetched at runtime
├── src/
│   ├── main.js                   # app entry: createApp(App).mount('#app'); imports leaflet CSS
│   ├── App.vue                   # the entire application (template/script/style)
│   └── assets/                   # bundled assets (currently a marker png)
├── babel.config.js
├── vue.config.js                 # SCSS modern API config
├── package.json
└── README.md
```

The application is essentially **one component**: `src/App.vue`. Nearly all
changes happen there.

## How `src/App.vue` works

- **Template** is written in **Pug** (`<template lang="pug">`). Mind indentation —
  Pug is whitespace-sensitive. The map is composed of `<l-*>` components from
  `@vue-leaflet/vue-leaflet`: `l-map`, `l-tile-layer`, `l-control-scale`,
  `l-geo-json`, `l-circle`, `l-circle-marker`, `l-marker`, `l-icon`, `l-tooltip`.
- **Map tiles** come from CARTO Voyager (`basemaps.cartocdn.com`), attributed to
  OpenStreetMap + CARTO.
- **`center`** (the 🏠 home marker) is a draggable Leaflet marker. On mount,
  `geolocate()` recenters it to the browser's current position.
- **`updatedCenter`** (the 🏃‍♀️ runner circle-marker) is updated live via
  `navigator.geolocation.watchPosition()` in `watchPosition()`; the watch is
  cleared in `beforeUnmount()`.
- **The 1 km circle** is an `l-circle` with `radius: 1000` (meters) drawn at
  `center`.
- **GeoJSON** of pedestrian streets is fetched in `created()` from
  `data/geojson/...` and stored in `this.geojson`; on fetch failure `showMap`
  is set to `false` so the map doesn't render with missing data.
- **Styling**: `styleFunction` (computed) returns the GeoJSON style; it
  intentionally reads `this.circle.fillColor` at the top to keep the Leaflet
  layer reactive. Emoji tooltips are enlarged via the `.leaflet-tooltip` SCSS
  rule.

### Key data fields (in `data()`)
`zoom`, `center`, `updatedCenter`, `circle` (the 1km radius styling),
`circlemarker`, `mapOptions`, `geojson`, `showMap`, `watchId`, tile `url`/`attribution`.

## Conventions

- **Stay minimal.** This is a one-component app; avoid introducing routers,
  state libraries, or build tooling unless the task genuinely requires it.
- **Template language is Pug, style language is SCSS** — keep new markup/styles
  in those languages to match the SFC.
- **Coordinates are `latLng(lat, lng)`** from Leaflet; radii are in **meters**.
- **Runtime geo data** (large GeoJSON) lives in `public/` and is `fetch`ed at
  runtime, NOT imported/bundled. Reference it via a URL relative to the origin.
- **Static images** referenced from the template (e.g. the marker icon) are
  served from `public/` by filename.
- Keep `npm run lint` clean (ESLint rules: vue3-essential + eslint:recommended).

## Git / workflow conventions

- Default branch is `main`. Dependency bumps come through **Dependabot** PRs and
  are merged via PR (see commit history — `chore(deps...)`).
- Use clear, conventional-style commit messages (e.g. `chore(deps): ...`,
  `feat: ...`, `fix: ...`) consistent with existing history.
- **Do not open a pull request unless explicitly asked.**
- Build output (`/dist`), `node_modules`, and local `.env*` files are gitignored.

## Gotchas

- The geolocation features (`getCurrentPosition` / `watchPosition`) require a
  **secure context (HTTPS or localhost)** and user permission — they won't work
  over plain HTTP on a remote host.
- The large GeoJSON file (~2 MB) is loaded on every page load; be mindful when
  changing how/when it's fetched.
- `index.html` includes a hardcoded Google Analytics tag (`UA-9706914-3`).
- SCSS uses the **modern Sass API** (configured in `vue.config.js`); keep that in
  mind if upgrading `sass`/`sass-loader`.
