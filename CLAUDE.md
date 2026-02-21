# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Compile TypeScript:**
```
tsc
```
Output goes to `nextrip/index.js`. There is no package.json, npm, or build pipeline — just `tsc` directly.

**Run locally:** Open `nextrip/index.html` in a browser. Use a URL hash to select a config (e.g., `index.html#lpm`). If no hash is provided, `lpm` is the default.

## Architecture

This is a zero-dependency, framework-free TypeScript app that displays a real-time transit departure board for Metro Transit (Minneapolis-St. Paul).

**Single source file:** `index.ts` compiles to `nextrip/index.js`. All application logic is in this one file — no modules, no imports.

**Config-driven:** Location configs are JSON files in `nextrip/` (e.g., `lpm.json`, `glen.json`). The URL hash selects which config to load at runtime. Each config defines:
- `name`: display name
- `weather`: NOAA grid coordinates (`officeId` + `gridpoints`)
- `stopGroups`: named groups (e.g., "Downtown", "Uptown"), each containing an array of stops with `id` (Metro Transit stop ID), `walkingTime` (minutes), and `routes` (route IDs to monitor)

**Data flow:**
1. `main()` loads the config JSON, then calls `updateDisplay()` on a 15-second interval
2. `updateDisplay()` fires parallel XHR requests for each stop group and for weather
3. All stop departures within a group are merged and sorted by `minutesMinusWalkingTime` (departure time minus walking time)
4. Departures filtered by: configured route IDs, non-`Skipped` schedule relationship, and `minutesMinusWalkingTime > 0`
5. HTML is rendered as strings and injected via `innerHTML`

**External APIs:**
- Metro Transit NexTrip: `https://svc.metrotransit.org/nextrip/{stop_id}`
- NOAA Weather: `https://api.weather.gov/gridpoints/{officeId}/{gridpoints}/forecast/hourly`

**Departure display:** Each row shows route, direction, "Departs" (minutes until departure or actual time), and "Leave" (minutes minus walking time). Clicking the "Departs" cell toggles between relative (`minutesUntilDepart`) and actual time (`departureTime`) formats — this calls `toggleDepartFormat()` which is a global function called from inline `onclick` handlers.

**Adding a new location:** Create a new JSON file in `nextrip/` following the same structure as `lpm.json`, then open `index.html#<filename-without-extension>` in a browser.
