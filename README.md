# SilviWeather — BC Silviculture Weather Alerts

A zero-dependency, single-file weather monitoring dashboard for BC silviculture operations. Tracks frost, heat stress, drought, flood, and wind conditions using real-time and historical data from [Open-Meteo](https://open-meteo.com/).

**No API keys. No build step. No server. Just GitHub Pages.**

---

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [User Guide](#user-guide)
   - [Dashboard Tab](#dashboard-tab)
   - [Daily History Tab](#daily-history-tab)
   - [Thresholds Tab](#thresholds-tab)
   - [Alert Log Tab](#alert-log-tab)
   - [Status Banner](#status-banner)
   - [Custom Location](#custom-location)
4. [Monitoring Stations](#monitoring-stations)
5. [Alert System](#alert-system)
   - [Frost](#frost)
   - [Heat Stress & VPD](#heat-stress--vpd)
   - [Drought](#drought)
   - [Flood](#flood)
   - [Wind](#wind)
6. [Default Thresholds](#default-thresholds)
7. [Open-Meteo API Integration](#open-meteo-api-integration)
   - [URL Structure & Parameters](#url-structure--parameters)
   - [Fields Reference](#fields-reference)
   - [Weather Code Reference](#weather-code-reference)
   - [VPD Explained](#vpd-explained)
8. [Data Storage (localStorage)](#data-storage-localstorage)
9. [Email Alerts (GitHub Actions)](#email-alerts-github-actions)
10. [Extending the App](#extending-the-app)
11. [Tech Stack](#tech-stack)
12. [License](#license)

---

## Overview

SilviWeather polls the Open-Meteo API on every page load, fetching 90 days of daily weather history plus a 3-day forecast for up to six monitoring stations simultaneously. It evaluates five alert types (frost, heat, drought, flood, wind) against user-configurable thresholds, displays per-station current conditions, and persists the alert history in browser localStorage.

The entire application is a single `index.html` file with no external CSS frameworks, no JavaScript libraries, and no build tooling.

---

## Quick Start

1. Fork this repository.
2. Enable GitHub Pages: **Settings → Pages → Source: `main`, folder: `/`**.
3. Visit `https://yourusername.github.io/SilviWeather/`.

The dashboard loads immediately and pulls live weather data on every visit. No configuration required.

**Local development:**

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Any static file server works — there is no build step and no package manager.

---

## User Guide

### Dashboard Tab

The default view. Shows:

- **Monitoring Stations** — a card grid listing all stations in the selected region with coordinates and elevation.
- **Current Conditions** — a table with one row per station showing current temperature, relative humidity, precipitation, wind speed, weather description, and any active alert badges.

Use the region selector in the header to switch regions. Click **↻ Refresh** to re-fetch data manually.

### Daily History Tab

Displays a scrollable table of daily weather data for a selected station. Columns:

| Column | Description |
|--------|-------------|
| Date | Calendar date (YYYY-MM-DD) |
| Max °C | Daily maximum temperature |
| Min °C | Daily minimum temperature |
| Precip mm | Total daily precipitation |
| ET₀ mm | Reference evapotranspiration (FAO-56 Penman-Monteith) |
| Wind km/h | Daily maximum wind speed |
| Flags | Alert badges for that specific day (Frost, Heat, Flood) |

A station selector dropdown in the table header lets you switch between stations in the region without a full reload. Data covers the past 90 days plus 3 forecast days.

### Thresholds Tab

Provides numeric inputs for every alert threshold. Changes take effect immediately on the current data when saved. Thresholds are written to `silvi-thresholds` in localStorage and persist across sessions.

Configurable values:

| Field | Input ID | Default |
|-------|----------|---------|
| Frost min temp | `thresh-frost-temp` | 1 °C |
| Heat max temp | `thresh-heat-temp` | 35 °C |
| Heat VPD | `thresh-heat-vpd` | 3.0 kPa |
| Drought consecutive days | `thresh-drought-days` | 14 days |
| Drought daily precip threshold | `thresh-drought-precip` | 1 mm |
| Flood daily | `thresh-flood-daily` | 50 mm |
| Flood 3-day cumulative | `thresh-flood-3day` | 75 mm |
| Wind max speed | `thresh-wind-speed` | 60 km/h |

**Save Thresholds** writes to localStorage and re-evaluates alerts on the current dataset. **Reset to Defaults** restores all values to factory defaults.

The tab also contains the email subscription form (see [Email Alerts](#email-alerts-github-actions)).

### Alert Log Tab

Shows a reverse-chronological list of up to 50 recent alert events (out of 100 stored). Each entry shows:

- Alert type badge (colour-coded)
- Timestamp (local browser time)
- Station name and alert detail (e.g., `Kamloops — Heat 37.2°C`)

Alerts are logged each time `fetchWeatherData()` runs and thresholds are exceeded. The log is capped at 100 entries in localStorage (`silvi-alerts`).

### Status Banner

Five cards across the top of the page provide an at-a-glance regional summary. Each card aggregates the **worst-case value across all stations** in the selected region:

| Card | Value Displayed | OK | Warn | Alert |
|------|-----------------|----|------|-------|
| **Frost Risk** | Lowest min temp across all stations (May–Sep only; "N/A" off-season) | ≥ frost threshold | — | < frost threshold |
| **Heat Stress** | Highest max temp across all stations | ≤ heat temp threshold | — | > heat temp threshold |
| **Drought Index** | Longest current dry streak across all stations | < 70% of day threshold | 70–99% of day threshold | ≥ day threshold |
| **Flood Risk** | Highest single-day precipitation across all stations | ≤ daily flood threshold | — | > daily flood threshold |
| **Wind** | Highest max wind speed across all stations | ≤ wind speed threshold | — | > wind speed threshold |

The Drought card is the only one with three states: green (ok), amber (warn, streak is 70–99% of the threshold), red (alert, streak ≥ threshold).

### Custom Location

Select **Custom Location** from the region dropdown to open a modal. Enter:

- **Location Name** — display label (defaults to "Custom" if blank)
- **Latitude** — decimal degrees north (48°–60°N for BC)
- **Longitude** — decimal degrees, negative for west (–140° to –114°)
- **Elevation** — metres above sea level (optional, used for display only)

Submitting creates a single-station region on the fly and fetches data immediately. Cancelling reverts the dropdown to the previously selected region.

---

## Monitoring Stations

Sixteen pre-configured stations across three BC Business Areas:

### Kamloops BA

| Station | Latitude | Longitude | Elevation |
|---------|----------|-----------|-----------|
| Kamloops | 50.70°N | 120.33°W | 345 m |
| Merritt | 50.11°N | 120.79°W | 594 m |
| Lillooet | 50.69°N | 121.94°W | 543 m |
| Clearwater | 51.65°N | 120.04°W | 421 m |
| Logan Lake | 50.50°N | 120.75°W | 1100 m |

### Okanagan BA

| Station | Latitude | Longitude | Elevation |
|---------|----------|-----------|-----------|
| Revelstoke | 50.98°N | 118.18°W | 445 m |
| Salmon Arm | 50.70°N | 119.27°W | 349 m |
| Vernon | 50.27°N | 119.27°W | 390 m |
| Kelowna | 49.88°N | 119.49°W | 433 m |
| Penticton | 49.48°N | 119.59°W | 344 m |

### Kootenays BA

| Station | Latitude | Longitude | Elevation |
|---------|----------|-----------|-----------|
| Grand Forks | 49.03°N | 118.44°W | 524 m |
| Castlegar | 49.32°N | 117.66°W | 495 m |
| Nelson | 49.50°N | 117.28°W | 553 m |
| Cranbrook | 49.50°N | 115.77°W | 920 m |
| Creston | 49.10°N | 116.51°W | 650 m |
| Invermere | 50.51°N | 116.03°W | 834 m |

---

## Alert System

Alerts are evaluated in `getStationAlerts()` using the most recent daily entry (index `len - 1`) from the Open-Meteo response, where `len = daily.time.length`. All threshold values come from the `thresholds` object (user-configurable, loaded from localStorage).

### Frost

**Condition:** Most recent day's minimum temperature is below the frost threshold, and the calendar month is May through September.

```js
const lastMinTemp = daily.temperature_2m_min[len - 1];
const month = new Date(daily.time[len - 1]).getMonth() + 1;  // 1-based
if (month >= 5 && month <= 9 && lastMinTemp < thresholds.frost.temp) {
    // FROST alert
}
```

Frost monitoring is disabled from October through April — planting seasons outside this window are not evaluated.

### Heat Stress & VPD

Two independent sub-checks, both tagged as `heat` type:

**Temperature check:** Most recent day's maximum temperature exceeds the heat threshold.

```js
const lastMaxTemp = daily.temperature_2m_max[len - 1];
if (lastMaxTemp > thresholds.heat.temp) {
    // HEAT (temp) alert
}
```

**VPD check:** The highest hourly vapour pressure deficit in the last 24 hourly records exceeds the VPD threshold.

```js
const last24vpd = hourly.vapour_pressure_deficit.slice(-24);
const maxVpd = Math.max(...last24vpd.filter(v => v !== null));
if (maxVpd > thresholds.heat.vpd) {
    // HEAT (VPD) alert
}
```

Both checks can fire independently on the same station. Alert badges are labelled separately: `Heat 37.2°C` and `VPD 3.4 kPa`.

### Drought

**Condition:** The current consecutive dry-day streak (counting backwards from the most recent day, stopping at the first day with precipitation ≥ the daily precip threshold) meets or exceeds the drought day threshold.

```js
let dryStreak = 0;
for (let i = len - 1; i >= 0; i--) {
    if (daily.precipitation_sum[i] < thresholds.drought.precip) {
        dryStreak++;
    } else break;
}
if (dryStreak >= thresholds.drought.days) {
    // DROUGHT alert
}
```

Maximum lookback is bounded by the API response length (90 days of history + 3 forecast days). When the streak reaches 90 days, the label reads `Dry 90d (90d max lookback)` to indicate the streak may extend further back than data availability allows.

### Flood

Two independent sub-checks, both tagged as `flood` type:

**Daily check:** Most recent day's total precipitation exceeds the daily flood threshold.

```js
const lastPrecip = daily.precipitation_sum[len - 1];
if (lastPrecip > thresholds.flood.daily) {
    // FLOOD (daily) alert — label: "82mm/day"
}
```

**3-day cumulative check:** Sum of the last three days' precipitation exceeds the 3-day threshold.

```js
const threeDay = daily.precipitation_sum.slice(-3).reduce((a, b) => a + (b || 0), 0);
if (threeDay > thresholds.flood.threeDay) {
    // FLOOD (3-day) alert — label: "95mm/3d"
}
```

### Wind

**Condition:** Most recent day's maximum wind speed exceeds the wind threshold.

```js
const lastWindSpeed = daily.wind_speed_10m_max[len - 1];
if (lastWindSpeed !== null && lastWindSpeed > thresholds.wind.speed) {
    // WIND alert
}
```

---

## Default Thresholds

| Alert | Parameter | Default | Unit | Notes |
|-------|-----------|---------|------|-------|
| Frost | Min temperature | 1 | °C | May–September only |
| Heat | Max temperature | 35 | °C | Year-round |
| Heat | VPD | 3.0 | kPa | Max hourly in past 24h |
| Drought | Consecutive dry days | 14 | days | Lookback up to 90 days |
| Drought | Daily precip threshold | 1 | mm | Days below this count as "dry" |
| Flood | Daily precipitation | 50 | mm | Single calendar day |
| Flood | 3-day cumulative | 75 | mm | Rolling 3-day sum |
| Wind | Max wind speed | 60 | km/h | Daily maximum |

---

## Open-Meteo API Integration

### URL Structure & Parameters

One API request is made per station on every data load. Requests are issued in parallel via `Promise.all`. Example URL for Kamloops:

```
https://api.open-meteo.com/v1/forecast?
  latitude=50.70&longitude=-120.33
  &daily=temperature_2m_max,temperature_2m_min,precipitation_sum,
         et0_fao_evapotranspiration,wind_speed_10m_max
  &hourly=vapour_pressure_deficit
  &current=temperature_2m,relative_humidity_2m,precipitation,wind_speed_10m,weather_code
  &past_days=90&forecast_days=3
  &timezone=America/Vancouver
```

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `latitude` / `longitude` | Station coords | Grid-point selection |
| `past_days` | 90 | Days of history before today |
| `forecast_days` | 3 | Days of forecast after today |
| `timezone` | `America/Vancouver` | All times in Pacific time |

### Fields Reference

**Daily fields** (`daily=...`):

| Field | Unit | Used For |
|-------|------|----------|
| `temperature_2m_max` | °C | Heat alert, history table |
| `temperature_2m_min` | °C | Frost alert, history table |
| `precipitation_sum` | mm | Drought/flood alerts, history table |
| `et0_fao_evapotranspiration` | mm | History table display |
| `wind_speed_10m_max` | km/h | Wind alert, history table |

**Hourly fields** (`hourly=...`):

| Field | Unit | Used For |
|-------|------|----------|
| `vapour_pressure_deficit` | kPa | Heat/VPD alert (last 24 values) |

**Current conditions** (`current=...`):

| Field | Unit | Displayed In |
|-------|------|--------------|
| `temperature_2m` | °C | Dashboard current conditions table |
| `relative_humidity_2m` | % | Dashboard current conditions table |
| `precipitation` | mm | Dashboard current conditions table |
| `wind_speed_10m` | km/h | Dashboard current conditions table |
| `weather_code` | WMO code | Dashboard (decoded to text description) |

### Weather Code Reference

WMO weather interpretation codes used by Open-Meteo and decoded in `getWeatherDesc()`:

| Code | Description |
|------|-------------|
| 0 | Clear |
| 1 | Mainly clear |
| 2 | Partly cloudy |
| 3 | Overcast |
| 45 | Fog |
| 48 | Rime fog |
| 51 | Light drizzle |
| 53 | Drizzle |
| 55 | Heavy drizzle |
| 61 | Light rain |
| 63 | Rain |
| 65 | Heavy rain |
| 71 | Light snow |
| 73 | Snow |
| 75 | Heavy snow |
| 80 | Rain showers |
| 81 | Moderate showers |
| 82 | Heavy showers |
| 85 | Snow showers |
| 95 | Thunderstorm |
| 96 | Thunderstorm w/ hail |

Any unrecognised code is displayed as `Code <N>`.

### VPD Explained

**Vapour Pressure Deficit (VPD)** is the difference between the amount of moisture the air could hold at saturation and the amount it actually holds. Higher VPD means drier air — plants transpire faster, stomata close, and photosynthesis slows.

- **< 1.0 kPa** — Low stress, good growing conditions
- **1.0–2.0 kPa** — Moderate stress
- **2.0–3.0 kPa** — High stress, visible wilting possible
- **> 3.0 kPa** — Very high stress; SilviWeather default alert threshold

Open-Meteo provides VPD as hourly data. SilviWeather takes the maximum value across the 24 most recent hourly entries.

---

## Data Storage (localStorage)

SilviWeather uses three localStorage keys. All are scoped to the page origin (your GitHub Pages domain).

### `silvi-thresholds`

User-configured alert thresholds. Written by **Save Thresholds** / **Reset to Defaults**. Read on every page load. If absent, `getDefaultThresholds()` provides the factory defaults.

```json
{
  "frost":   { "temp": 1 },
  "heat":    { "temp": 35, "vpd": 3.0 },
  "drought": { "days": 14, "precip": 1 },
  "flood":   { "daily": 50, "threeDay": 75 },
  "wind":    { "speed": 60 }
}
```

### `silvi-alerts`

Ring buffer of alert events, capped at 100 entries. New alerts are prepended; the array is sliced to 100 after each append. Written each time `analyzeAlerts()` detects active alerts.

```json
[
  {
    "time": "2026-07-15T18:42:00.000Z",
    "station": "Kamloops",
    "type": "heat",
    "label": "Heat 37.2°C"
  },
  {
    "time": "2026-07-15T18:42:00.000Z",
    "station": "Merritt",
    "type": "drought",
    "label": "Dry 18d"
  }
]
```

`type` is one of: `frost`, `heat`, `drought`, `flood`, `wind`.

### `silvi-subscribers`

Array of email addresses entered via the Thresholds tab subscription form. Written client-side only — actual email delivery requires a GitHub Actions workflow to read this key server-side (see [Email Alerts](#email-alerts-github-actions)).

```json
["ops@example.com", "forester@example.gov.bc.ca"]
```

---

## Email Alerts (GitHub Actions)

The dashboard stores subscriber emails in localStorage, but delivery requires a server-side process. The recommended approach is a scheduled GitHub Actions workflow that:

1. Fetches weather data from Open-Meteo for each configured region
2. Evaluates the same alert logic
3. Reads the subscriber list from a config file (not localStorage, which is browser-only)
4. Sends emails via SMTP when thresholds are exceeded

### Workflow Template

```yaml
name: Daily Weather Check
on:
  schedule:
    - cron: '0 14 * * *'  # 6 AM Pacific daily
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: node .github/scripts/check-weather.js
        env:
          SMTP_HOST: ${{ secrets.SMTP_HOST }}
          SMTP_USER: ${{ secrets.SMTP_USER }}
          SMTP_PASSWORD: ${{ secrets.SMTP_PASSWORD }}
```

Add repository secrets (`SMTP_HOST`, `SMTP_USER`, `SMTP_PASSWORD`) under **Settings → Secrets and variables → Actions**. The check-weather script is not included — implement it to mirror the alert logic in `index.html` using Node's `fetch` and an SMTP library such as `nodemailer`.

---

## Extending the App

### Adding a Region or Station

Edit the `REGIONS` constant near line 632 in `index.html`. Each region requires a `name` (display label) and a `stations` array. Each station requires:

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name |
| `lat` | number | Decimal latitude (°N) |
| `lon` | number | Decimal longitude (negative for °W) |
| `elevation` | number | Metres above sea level |

```javascript
'my-region': {
    name: 'My Region BA',
    stations: [
        { name: 'My Station', lat: 53.92, lon: -122.75, elevation: 680 },
        { name: 'Second Station', lat: 54.10, lon: -123.10, elevation: 720 },
    ]
}
```

Also add a matching `<option>` to the `<select id="regionSelect">` element in the HTML.

### Adding a New Alert Type

1. Add the threshold fields to `getDefaultThresholds()` and the `saveThresholds()` / `populateThresholdInputs()` functions.
2. Add corresponding `<input>` elements in the Thresholds tab HTML.
3. Add the detection logic to `getStationAlerts()`.
4. Add a status card to the HTML status banner and update `analyzeAlerts()` to populate it.
5. Add a CSS colour variable if needed (current palette: `--frost`, `--heat`, `--drought`, `--flood`, `--wind`).

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Single HTML file — vanilla JS (ES2020+), no frameworks |
| Styling | CSS custom properties, dark theme |
| Data | [Open-Meteo API](https://open-meteo.com/) — free, no API key |
| Hosting | GitHub Pages — free for public repositories |
| Persistence | Browser `localStorage` — no backend database |
| Email alerts | GitHub Actions + SMTP (optional, not included) |

**Open-Meteo free tier:** No API key required. Up to 10,000 requests per day. With 16 stations across 3 regions, a full refresh is 5–6 API calls (one per station in the selected region). Global coverage, ~1 km resolution, ERA5 reanalysis for history.

---

## License

MIT
