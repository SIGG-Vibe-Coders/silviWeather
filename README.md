# SilviWeather — BC Silviculture Weather Alerts

A simple, free weather monitoring dashboard for BC silviculture operations. Tracks frost, heat stress, drought, and flood conditions using real-time data from [Open-Meteo](https://open-meteo.com/).

**No API keys required. No server needed. Just GitHub Pages.**

## Features

- 🌡️ **Real-time conditions** from multiple weather stations per region
- ❄️ **Frost alerts** — Temperature drops below threshold (May–September)
- 🔥 **Heat stress** — Max temperature and Vapour Pressure Deficit (VPD) monitoring
- ☀️ **Drought tracking** — Consecutive dry days with configurable precipitation threshold
- 🌊 **Flood risk** — Daily and 3-day cumulative precipitation alerts
- 📊 **30-day history** with daily breakdowns per station
- ⚙️ **Configurable thresholds** saved to browser
- 📧 **Email alert subscriptions** (with GitHub Actions — see below)

## Quick Start

1. Fork this repo
2. Enable GitHub Pages (Settings → Pages → Source: `main`, folder: `/`)
3. Visit `https://yourusername.github.io/silviculture-weather-alerts/`

That's it. The dashboard pulls live data from Open-Meteo on every page load.

## Regions

Pre-configured monitoring stations:

| Region | Stations |
|--------|----------|
| Kamloops TSA | Kamloops, Merritt, Salmon Arm, Clearwater, Logan Lake |
| Prince George TSA | Prince George, Vanderhoof, Quesnel, McBride |
| Williams Lake TSA | Williams Lake, 100 Mile House, Alexis Creek |

### Adding Custom Stations

Edit the `REGIONS` object in `index.html` to add your own stations. You just need latitude, longitude, and elevation:

```javascript
'my-region': {
    name: 'My Region',
    stations: [
        { name: 'Station Name', lat: 50.0, lon: -120.0, elevation: 500 },
    ]
}
```

## Default Alert Thresholds

| Alert | Threshold | Notes |
|-------|-----------|-------|
| Frost | Min temp < 1°C | May–September only |
| Heat | Max temp > 35°C | — |
| Heat (VPD) | VPD > 3.0 kPa | High transpiration stress |
| Drought | 14+ days < 1mm precip | Consecutive dry days |
| Flood (daily) | > 50mm in one day | — |
| Flood (3-day) | > 75mm cumulative | Rolling 3-day window |

All thresholds are configurable in the dashboard and saved to your browser's local storage.

## Email Alerts (GitHub Actions)

To enable automated daily email checks:

1. Create `.github/workflows/weather-check.yml` (template below)
2. Add repository secret `SMTP_PASSWORD` for your email provider
3. Configure subscriber list in `subscribers.json`

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

## Data Source

All weather data comes from [Open-Meteo](https://open-meteo.com/), a free and open-source weather API:

- No API key required
- 10,000 requests/day on free tier
- Hourly and daily data with 30-day history
- Global coverage with ~1km resolution
- Variables: temperature, precipitation, VPD, wind, ET₀, and more

## Tech Stack

- **Frontend:** Single HTML file, vanilla JS, no dependencies
- **Data:** Open-Meteo API (free, no auth)
- **Hosting:** GitHub Pages (free)
- **Alerts:** GitHub Actions (optional, free for public repos)

## Contributing

PRs welcome! Ideas for expansion:
- More BC regions and stations
- Fire Weather Index (FWI) calculations
- Snow depth and snowmelt tracking
- Integration with BC Wildfire Service data
- Mobile-optimized views
- Chart visualizations (precipitation trends, temperature graphs)

## License

MIT
