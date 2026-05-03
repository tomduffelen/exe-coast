# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Exe Coast is a water sports conditions forecaster for coastal locations in Devon, UK (Exmouth and Budleigh Salterton). It is a **zero-dependency, single-file SPA** — the entire application lives in `index.html`.

## Running the App

No build step. Open directly in a browser or serve statically:

```
python -m http.server 8080
# or
npx http-server .
```

There are no tests, no linting config, and no package.json.

## Architecture

Everything is in `index.html` (~880 lines), structured in three sections:

1. **CSS** (lines 8–206): Design tokens via CSS custom properties (`--c-ink`, `--c-ocean`, `--c-teal`, status colors `--c-ideal/good/warn/danger`). Responsive grid layout.
2. **HTML** (lines 208–335): Two screens — a location picker overlay and the main app (metrics, sport cards, tide strip, forecasts, AI summary, safety warnings).
3. **JavaScript** (lines 337–882): All logic inline, no modules.

### Key Data Structures

- **`LOCS`**: Object keyed by location slug (`exmouth`, `budleigh`) with lat/lon coordinates and display names.
- **`SPORTS`**: Array of sport configs (`kayak`, `kitesurf`, `sup`, `dinghy`) used to drive the rating cards.

### Core Logic Flow

1. User selects location → `loadDay(loc, dayOffset)` is called.
2. Fetches from three APIs in parallel (weather, marine, tides) with LocalStorage caching.
3. `getRating(sport, wind, gust, waveHeight, wavePeriod, windDir, loc)` → `'ideal'|'good'|'warn'|'danger'` for each sport.
4. `getFlags(conditions)` → array of safety alerts (offshore wind, gusty conditions, spring tides, etc.).
5. Cloudflare Worker at `exe-coast-api.tomduffelen.workers.dev` is called for an AI-generated natural-language summary (Claude API, server-side to protect the key).
6. DOM is updated directly (no virtual DOM).

### API Integrations

| API | Purpose | Auth |
|-----|---------|------|
| Open-Meteo | Hourly wind, gusts, rain, wave height/period | None (free) |
| WorldTides | Tide heights and extremes | API key in JS |
| Cloudflare Worker | Claude AI summaries | None from client |

### Caching (LocalStorage)

- Weather/marine data: 30-minute TTL
- Tide data: 720-minute (12h) TTL
- AI summaries: keyed per `{location}-{date}`, no expiry

### Adding a New Sport

1. Add an entry to the `SPORTS` array with `id`, `name`, `icon`.
2. Add a case in `getRating()` with wind/wave thresholds specific to the sport.
3. Add an advice function (e.g., `getKayakAdvice()`) returning context-aware strings.

### Adding a New Location

Add to `LOCS` with `lat`, `lon`, `name`, and a slug key. The location picker renders all entries in `LOCS` automatically.
