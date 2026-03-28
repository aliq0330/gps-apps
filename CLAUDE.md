# CLAUDE.md — GPS Apps Codebase Guide

## Project Overview

This repository contains a standalone mobile GPS/weather web application built as a single HTML file with embedded CSS and JavaScript. No build system, no dependencies, no frameworks — just open the file in a browser.

**Single application:**
- `gpss.html` — Uses Yandex Maps API for reverse geocoding (requires API key)

**Target audience:** Turkish-speaking mobile users (`tr-TR` locale, labels in Turkish)

---

## Repository Structure

```
gps-apps/
├── gpss.html      # Main app — Yandex Maps geocoding variant
└── README.md      # Minimal placeholder
```

No `package.json`, no `node_modules`, no build config, no test files, no CI/CD.

---

## Architecture

The file is a self-contained single-page application:

```
<html>
  <head>
    CSS (embedded <style>) — ~150 lines
  </head>
  <body>
    HTML structure — ~100 lines
    JavaScript (embedded <script>) — ~430 lines, 67 functions
  </body>
</html>
```

### Global State

All runtime state lives in a single object `S`:

```javascript
S = {
  lat, lng, alt,        // GPS coordinates & altitude
  accuracy,             // GPS accuracy in meters
  heading, speed,       // GPS heading and speed
  watchId,              // navigator.geolocation.watchPosition ID
  compassHeading,       // Device orientation heading
  compassActive,        // Boolean — compass running
  compassListener,      // Stored event listener reference
  addressCity,          // Extracted city name from geocoding
  weatherData           // Full Open-Meteo API response
}
```

### External APIs

| Service | API | Auth |
|---|---|---|
| Geolocation | Browser native `navigator.geolocation` | User permission |
| Reverse geocoding | Yandex Maps `geocode-maps.yandex.ru/1.x/` | API key embedded in code |
| Weather | Open-Meteo `api.open-meteo.com/v1/forecast` | None required |
| Map tiles | OpenStreetMap `tile.openstreetmap.org` | None required |
| Fonts | Google Fonts (DM Sans, Space Mono) | None required |

---

## Key Conventions

### Naming Patterns

- **State object:** Single global `S` holds all mutable app state
- **Private variables:** Prefixed with `_` (e.g., `_theme`, `_mq`, `_ms`)
- **Abbreviated functions:** Short names are intentional (e.g., `wI()` = weather icon, `dms()` = decimal-to-DMS, `bR()` = button reset, `upCC()` = update compass card)
- **DOM IDs:** Kebab-case (e.g., `#coord-card`, `#weather-section`)

### Code Style

- **Minified, compressed** production-style code — no whitespace formatting
- **No external dependencies** — everything inline, no imports
- **Turkish strings** — all user-visible labels, units, and messages are in Turkish
- **Direct DOM manipulation** — `document.getElementById`, `innerHTML`, `classList`
- **Mobile-first CSS** — max-width 430px, touch-optimized

### Function Categories

| Category | Functions |
|---|---|
| Location | `getLocation()`, `onPos(pos)`, `onErr(err)`, `refreshLocation()`, `resetBtn()` |
| Geocoding | `fetchAddr()` — Yandex hierarchical `kind` component system |
| Weather | `fetchWeather()`, `openDayDetail(idx)`, `renderDay(idx)`, `wI(code)`, `wD(code)` |
| Map | `drawMap()`, `openMaps()` |
| Compass | `startCompass()`, `drawCompass(h)`, `animC()`, `onDO(event)` |
| UI/Modal | `openModal(id)`, `closeModal(id)`, `_attachSwipe(id)`, `navDay(dir)` |
| Display | `fillTimeModal()`, `upSun()`, `upExt()`, `upCC()`, `buildDots(n,a)` |
| Utilities | `shareAs(type)`, `copyCoord(type)`, `copyRowValue(el,text)`, `dms(deg,type)`, `sunT(lat,lng,date)`, `utc()`, `showToast(msg)`, `_applyTheme(name)`, `flash(el,t)` |

---

## Features

- **Real-time GPS tracking** via `navigator.geolocation.watchPosition()` (high accuracy, 15s timeout)
- **Reverse geocoding** — Yandex Maps API converts coordinates to human-readable address
- **10-day weather forecast** — Open-Meteo API with Turkish weather descriptions
- **Compass** — uses `DeviceOrientationEvent`, requires iOS 13+ permission prompt
- **OSM map canvas** — renders OpenStreetMap tiles on an HTML `<canvas>`
- **Sunrise/sunset** — calculated locally via `sunT()` (no external API)
- **Dark/Light theme** — auto-detects via `prefers-color-scheme`, saved to `localStorage`
- **Location sharing** — Web Share API + Google Maps deep links
- **Coordinate copy** — Clipboard API with toast feedback
- **Swipe gestures** on modals — touch event handling via `_attachSwipe()`

---

## Development Workflow

### Making Changes

Since there is no build step, edit `gpss.html` directly and open in a browser to test:

```bash
# Open in browser (Linux)
xdg-open gpss.html

# Or serve locally for HTTPS (required for geolocation on some browsers)
python3 -m http.server 8080
# Then visit http://localhost:8080/gpss.html
```

> **Note:** GPS geolocation and DeviceOrientation require HTTPS in production. Use a local server or `localhost` during development.

### Testing

No automated tests exist. Manual testing checklist:
- [ ] GPS location permission prompt appears
- [ ] Coordinates update when position changes
- [ ] Address loads after GPS fix
- [ ] Weather data loads (10-day forecast)
- [ ] Map canvas renders OSM tiles
- [ ] Compass activates (device orientation events)
- [ ] Theme toggle works (dark/light/system)
- [ ] Modals open/close with swipe gesture
- [ ] Share and copy buttons produce correct output

### Branching

Development branch: `claude/add-claude-documentation-w7fWn`

---

## Security Notes

- **Yandex API key** is hardcoded in `gpss.html` — do not expose in public repositories or rotate if compromised
- **GPS data** is sensitive — the app does not transmit coordinates anywhere except to the geocoding/weather APIs
- **Permissions required:** Geolocation, Device Orientation (iOS 13+), Clipboard (context-dependent)
- No server-side component — entirely client-side, no user data stored remotely

---

## Browser Compatibility

**Required APIs:**
- Geolocation API
- Fetch API
- Canvas API
- Web Storage (localStorage)
- CSS Custom Properties

**Optional APIs (graceful fallback):**
- DeviceOrientationEvent (compass)
- Clipboard API (copy feature)
- Web Share API (native share)

**Target:** Mobile Safari (iOS), Chrome Mobile (Android), Firefox Mobile
