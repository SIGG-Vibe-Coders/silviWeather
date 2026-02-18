# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SilviWeather is a zero-dependency, single-file weather monitoring dashboard for BC silviculture operations. The entire application lives in `index.html` — no build step, no package manager, no server required. It is hosted on GitHub Pages.

## Development

To develop locally, serve `index.html` with any static file server:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no build, lint, or test commands. Changes to `index.html` are immediately reflected when the page is refreshed.

## Architecture

`index.html` is organized into three co-located sections:

- **CSS** (lines ~7–411): Dark-theme styling using CSS custom properties. Alert colors: frost = blue, heat = orange, drought = yellow, flood = teal, wind = slate.
- **HTML** (lines ~412–625): Four tabs — Dashboard, Daily History, Thresholds, Alert Log. Status banner shows 4 aggregated metrics (including wind).
- **JavaScript** (lines ~626–1179): Vanilla JS, no framework. Structured as distinct functional modules:

  | Module | Responsibility |
  |--------|----------------|
  | `REGIONS` constant | Pre-configured BC monitoring stations (lat/lon/elevation) |
  | `loadThresholds` / `saveThresholds` | Persist user alert thresholds to localStorage |
  | `fetchWeatherData()` | Parallel `Promise.all` fetches from Open-Meteo API |
  | `getStationAlerts()` / `analyzeAlerts()` | Alert logic; aggregates worst-case across stations |
  | `render*()` functions | DOM updates for each tab view |

## Data Flow

```
User selects region → fetchWeatherData() → Open-Meteo API (past_days=30, forecast_days=3)
  → analyzeAlerts() → renderCurrentConditions() / renderHistoryTable()
  → Alert history saved to localStorage
```

**Open-Meteo fields used:** `temperature_2m_max`, `temperature_2m_min`, `precipitation_sum`, `et0_fao_evapotranspiration`, `wind_speed_10m_max`, `vapour_pressure_deficit` (hourly), plus current conditions. Timezone is always `America/Vancouver`.

## LocalStorage Keys

- `silvi-thresholds` — user-configured alert thresholds
- `silvi-alerts` — alert history (capped at 100 entries)
- `silvi-subscribers` — email subscription list (requires GitHub Actions for delivery)

## Alert Logic

| Alert | Default Trigger | Season Gate |
|-------|----------------|-------------|
| Frost | Min temp < 1°C | May–Sept only |
| Heat stress | Max temp > 35°C OR VPD > 3.0 kPa | None |
| Drought | 14+ consecutive days < 1mm precipitation | None |
| Flood | Single day > 50mm OR 3-day total > 75mm | None |
| Wind | Max daily wind speed > 60 km/h | None |

All thresholds are user-configurable via the Thresholds tab and stored in localStorage.

## Extending the App

To add a new region or station, edit the `REGIONS` object near line 632 in `index.html`. Each station requires `name`, `lat`, `lon`, and `elevation`.

Email alert delivery requires a GitHub Actions workflow (not included) that reads `silvi-subscribers` and calls the Open-Meteo API server-side.
