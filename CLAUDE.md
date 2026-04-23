# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of static, self-contained HTML pages for "north by north" activity caravans in 道北（北海道北部）. Each program gets its own directory tree under `nbn/<program>/<view>/index.html`. Currently only `nbn/caravan-1/map/index.html` exists — an interactive Leaflet-based itinerary map for the 2026-04-24〜26 caravan.

There is no build system, no package manager, no tests. Each HTML file is standalone: CSS and JS are inlined, external libs come from CDNs (Leaflet, CARTO tiles, Google Fonts).

## Commands

Open locally in a browser:

```
open nbn/caravan-1/map/index.html
```

No build, lint, or test commands exist. Changes are verified by loading the file in a browser.

## Architecture of the map page

Everything in `nbn/caravan-1/map/index.html` is driven by a single `stops` array inside the inline `<script>`. Each stop has:

- `id`, `day` (1–3), `seq` (order within a day), `time`, `title`, `subtitle`, `note`
- `lat`, `lng` — used both for map placement and for the "Google Maps で開く" link (coordinate-based, not place-ID based; this was intentional — see commit 26d41f6)
- `optional: true` for 温泉 stops, which get dashed border + `--steam` color and can be toggled off
- `placeId` is stored but currently unused for link generation; don't switch the link back to `place_id=...` format without checking history

Derived from the array, in order:
1. Markers (`L.divIcon` with day-colored teardrop pins; seq number inside)
2. Per-day dashed polyline connecting non-optional stops in seq order, colored `--forest`/`--gold`/`--copper` for days 1/2/3
3. Thin steam-colored connectors from each optional stop to its seq-adjacent non-optional neighbors
4. Filter chips (all / day 1 / 2 / 3) and the 温泉 toggle, both re-running `applyFilters()` which rebuilds visible markers, route lines, and `fitBounds`
5. The bottom detail panel (`#detailPanel`), shown on marker click, dismissed by map click, ×, or Esc

## Editing conventions

- **Adding or moving a stop** = edit the `stops` array. `day` + `seq` determine marker number and route connectivity; `optional: true` changes rendering and filtering. Coordinates drive both the pin and the Google Maps link, so verify them before committing.
- **Canonical program name** is 道北アクティビティキャラバン (not 〜キャンプ, not 〜ツアー). There has been prior churn on this; don't rename without an explicit ask.
- **Privacy posture** is deliberate: `<meta name="robots" content="noindex,nofollow">` is set, and the footer asks guests not to publish specific activity locations. Don't weaken either without discussion — the footer text is part of the product, not boilerplate.
- CSS custom properties at the top of `<style>` (`--forest`, `--gold`, `--copper`, `--steam`, etc.) are the single source of truth for the palette. Day colors appear in CSS, in the `dayColors` JS object, and in marker/tag `[data-day]` selectors — keep them in sync.
