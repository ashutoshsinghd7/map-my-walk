# Walking Route Planner (working title)

A small mobile app for planning a walking route by tapping points on a map, then walking it and seeing your actual GPS trail drawn on the same map.

> **Status:** Concept / pre-prototype. Nothing is built yet. This README describes the plan; see [`docs/research.md`](docs/research.md) for the evidence behind it and what is still unproven.

## What it does (V1)

1. Open the map
2. Tap points to define your intended route
3. The app generates a route snapped to walkable roads and trails
4. Start your walk
5. The app reads your GPS position
6. Your actual trail is drawn on the map
7. Stop the walk

## What V1 deliberately does not do

Login, backend, cloud sync, social features, calorie counting, fitness analytics, AI, perfect GPS, GPS smoothing, off-route alerts, turn-by-turn navigation, route history, complex route editing, and offline maps.

Keeping these out is a design choice to keep the first version small. One visible consequence: without GPS smoothing, the drawn trail will show raw GPS noise.

## Roadmap

| Version | Goal | Contents |
|---|---|---|
| V0 | Map experiment | Map renders, user taps points, points are displayed |
| V1 | Walking prototype | Tap points, generate route, start walk, GPS trail displayed |
| V1.1 | Route following | Planned route shown, distance from planned route, haptic off-route warning |
| V2 | Reliability | GPS filtering, background tracking, battery optimization, route saving |

The roadmap is a plan, not a commitment.

## Why this exists, and an honest caveat

Apps such as Footpath, TouchTrails and Komoot already let you plan walking routes, and some of their features are paid (for example turn-by-turn navigation and offline maps). Their free tiers already cover a lot of what V1 does, including snapping to paths in Footpath's case. So this project is not claiming an unmet market gap. It is a focused, simple, open-source take on route planning plus trail recording. Details and sources are in [`docs/research.md`](docs/research.md#3-existing-solutions).

## Privacy

There is no account and no backend of our own. That does not make the app fully offline: your tapped waypoints are sent to a third-party routing service to get a walkable route, and map tiles are fetched from a tile provider, which reveals the area you are viewing. Our design intent is that your recorded GPS trail stays on your device; this needs to be verified once the app exists.

## Technology (planned)

| Area | Choice |
|---|---|
| App framework | Flutter or React Native (undecided) |
| Map rendering | MapLibre |
| Routing | Valhalla (pedestrian) or OSRM (foot), to be tested |
| Map tiles | Not yet chosen |

Free public demo routing and tile servers have usage restrictions that may not allow a public release. See [`docs/research.md`](docs/research.md#5-services-and-usage-policies-the-main-v1-risk).

## Next steps

1. V0 spike: map renders, user taps points, points are displayed.
2. Test pedestrian routing for tapped waypoints on routes we actually walk.
3. Walk with raw GPS updates to see how noisy the trail is.
4. Choose tile and routing providers for release.

## Open questions

Target platform(s), framework choice, release providers and hosting, intended audience (personal, open-source learning, or public release), and repository licence. See [`docs/research.md`](docs/research.md#9-open-questions).

## Documentation

- [`docs/research.md`](docs/research.md): research notes with sources, claim status labels and a change log

## Licence

To be decided.