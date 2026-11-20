# doot — Trip Planner

A single-page trip planner built on the Google Maps JavaScript API, styled like Google Maps.
Everything lives in **one file, `index.html`** (~11,000 lines: CSS, HTML and JS inline). No build step,
no dependencies to install, no server. Keep it a single file unless the owner asks otherwise.

The owner works mostly from a phone and has been updating the repo by uploading `index.html`
through the GitHub website (hence the "Add files via upload / Delete index.html" history).
Explain steps in plain, non-technical language.

## What the app does
- **Map & search**: Google map with a pill search bar, category chips (restaurants, etc.),
  results list, place panel with tabs, dropped-pin card, Street View, map-type and location controls.
- **Directions**: driving / transit / walking / cycling (+ taxi priced as driving), route cards,
  transit cards, alternative routes, transit stop panel with departures.
- **Trip planner** (the left panel, `#itin-panel`), three screens switched by `showScreen(name)`:
  - `#screen-home` – folders (trips) with colour + emoji (`openNewFolderModal`)
  - `#screen-folder` – the days in a trip (`goToFolder`)
  - `#screen-day` – a day itinerary: ordered **stops** joined by **legs** (one mode per leg),
    arrival/departure times cascading from a start time + dwell durations, drag-to-reorder,
    pick-a-point-from-map mode, per-leg route options, cost/time/distance totals, elevation
    estimate for walk/cycle, export as text.
- **AI planner** (tab on the day screen): asks Claude to draft a day itinerary (pace, budget,
  attractions, free text), streams the reply, shows variants, `acceptAIItinerary()` turns
  the chosen variant into stops/legs.
- **Live mode** (`startLiveMode` … `endLiveMode`): step-by-step day-of navigation with a
  countdown, GPS tracking (`_gpsOnPosition`), auto-advance at stops, off-route detection and
  reroute, "leaving now" / "arrived" buttons, live time cascading, map dimming of done legs.
- Mobile layout (portrait): full-screen panel with a map toggle (`initMobile`, `toggleMobileMap`).

## Code map (search for these names; line numbers drift)
- CSS: top `<style>` block, sections marked `/* ══ … ══ */` (left panel, directions, place panel,
  transit stop panel, itinerary, home/folder/day screens, AI pane, API-key banner, mobile).
- HTML: map + panels, then the three `itin-screen` divs, then modals.
- JS (one big `<script>`), roughly in order:
  `initMap` (Google callback) → search/autocomplete/results → place panel & add-to-trip menu
  → directions & route rendering → itinerary core (`itinStops`, `itinLegs`, `itinRenderers`,
  leg fetching, time cascade, summary cards, totals) → map-select mode → folders/days storage
  & screen navigation → AI planner → API-key handling → mobile → live mode & GPS → leg mode chips.
- Google Maps script tag is at the very bottom (`…&libraries=places,geometry&callback=initMap`).

## Data (browser localStorage, per device)
- `folders_index` → `[{id, name, emoji, colour, createdAt}]` (`getFolders` / `saveFolders`)
- `folder_<id>` → `[day]`, day = `{id, title, date, savedAt, stops:[], legs:[]}` (`saveFolderDays`, `saveDayNow`)
  - stop: `{id, name, place, arrivalTime, depTime, durationMins, notes}`
  - leg: `{id, mode, waypoints, result, color, durationSecs, distanceM}`
- `anthropic_api_key` → the user's own Claude API key (entered in the app; never commit one).

## External services
- **Google Maps JS API** (Places, Directions, Geocoding, Geometry, Street View). The API key is in
  the script tag and therefore public: it must be restricted in Google Cloud Console
  (HTTP referrers = the site's domain, and only the Maps JS / Places / Directions / Geocoding APIs).
- **Anthropic Messages API**, called straight from the browser with the user's key
  (`anthropic-dangerous-direct-browser-access` header). The model ids in the code are older
  ones; check current model ids before changing them.

## Working on it
- Edit `index.html` directly. Keep changes surgical; the file is large, so search by function
  name rather than reading it whole.
- To test: serve the folder (`python3 -m http.server`) and open it in a browser (Playwright is
  available in Claude Code cloud sessions). Google Maps only loads from referrers the API key
  allows, and needs network access to `maps.googleapis.com`, so `localhost` may need adding to
  the key's allowed referrers for local testing.
- The owner's separate project, London live trains, is in the `finncoates/london-live` repo, not here.
