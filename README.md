# Map My Walk

A privacy-first mobile app for planning a walking route **before** you walk, then tracking progress along that route with GPS and haptic feedback.

Most fitness apps only record where you went after the fact. Map My Walk is proactive: draw the path you want, pocket the phone, and walk. No accounts, no social feed, no calorie dashboards — just a lightweight “zen walker” companion.

## How it works

1. Open the map and draw a rough path with your finger.
2. A routing engine map-matches the drawing onto real walkable roads and trails.
3. Start the walk; the app locks onto your GPS location.
4. A background service keeps you inside an “invisible tube” (buffer) around the planned route.
5. If you drift outside the tube, the app sends a haptic alert.

## Goals

- Frictionless “zero-tap launch” — open, draw, walk; no login required
- Purely on-device; privacy-first with no backend for V1
- Focus on custom route planning and on-route tracking only

## MVP

- Interactive map to draw a route
- Map matching (snap drawing to real paths)
- Background GPS tracking
- Off-route haptic alert
- Local route storage (no accounts / no cloud)

**Out of scope for V1:** backend, social features, calorie counters, leaderboards, user accounts.

## Tech stack (proposed)

| Area | Choice | Notes |
|------|--------|--------|
| Frontend | Flutter or React Native | Cross-platform iOS + Android |
| Backend | None (local / serverless) | Simple, private, zero host cost |
| Local storage | SQLite / AsyncStorage / SharedPreferences | Save drawn routes on device |
| Map renderer | MapLibre GL | Open-source alternative to Google Maps / Mapbox |
| Routing / map matching | Valhalla or OSRM | Free/public instances for snapping paths |
| Spatial math | Turf.js (or Dart equivalent) | Distance + buffer / off-route checks |
| Location | `react-native-geolocation-service` or Flutter `geolocator` | GPS capture |

AI/ML is not needed for the MVP.

## Main technical challenges

- **Map matching** — Finger strokes are screen coordinates; they must be snapped to legal walking paths so the line doesn’t cut through buildings.
- **GPS drift / jitter** — Signal bounce under trees or near buildings; needs smoothing so wild jumps don’t break tracking.
- **Background OS kills** — iOS and Android aggressively kill background GPS; correct permissions and background handlers are critical.
- **Off-route detection** — Continuously check that the smoothed GPS point stays inside a buffer polygon around the route, allowing for normal GPS error.

## Competitive landscape

| App | Gap vs Map My Walk |
|-----|--------------------|
| **Footpath Route Planner** | Closest competitor; core turn-by-turn features often behind a premium paywall |
| **TouchTrails** | Similar draw-and-track utility, but free tier heavily limited (e.g. few saved routes) |
| **Komoot** | Powerful but complex; account + community features — leaves room for a simpler tool |

## Next step

Spin up a basic React Native or Flutter sandbox with MapLibre, send a string of coordinates to a free OSRM or Valhalla instance, and validate map matching on local roads before building the rest of the app.

## Research

Full notes: [project_research_note.md](./project_research_note.md)
