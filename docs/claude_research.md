# Research Notes: Walking Route Planner

> **Status:** Concept / pre-prototype. Nothing is built yet.
> **Evidence last checked:** 2026-10-02. Prices, free-tier limits and usage policies change; re-check before relying on them.
> **Purpose of this file:** Record what we looked at, what we found, and what is still unproven, so a first-time reader (or future us) can tell facts from assumptions.

## How to read this document

Every non-trivial claim carries one of three labels:

| Label | Meaning |
|---|---|
| **[Sourced]** | Backed by a linked source, checked on the date above. |
| **[To test]** | A hypothesis we plan to verify with a small experiment. |
| **[Assumption]** | A design decision or belief with no evidence yet. |

---

## 1. Summary

- We want a small mobile app: tap points on a map, get a route snapped to walkable paths, walk it, and see the actual GPS trail drawn on the map. **[Assumption]** that this is useful to enough people to matter.
- V1 deliberately excludes accounts, a backend of our own, cloud sync, social features, fitness metrics, AI, GPS smoothing, off-route alerts, turn-by-turn navigation, route history, complex route editing, and offline maps.
- The V1 feature set **overlaps heavily with what existing apps already offer free** (see section 3). We do not have evidence of a large unmet market gap. The honest framing today is a focused, simple, open-source project, not a proven gap.
- Routing and map tiles come from third-party free services whose usage policies limit production use (section 5). Choosing sustainable providers is the main open technical question for V1.

## 2. Scope

### V1 user flow

1. Open map
2. Tap points to define the intended route
3. Generate/snap route to walkable roads and trails
4. Start walk
5. Get GPS position
6. Draw actual trail on map
7. Stop walk

### Not in V1

Login, backend, cloud sync, social, calories, fitness analytics, AI, perfect GPS, GPS smoothing, off-route alerts, turn-by-turn navigation, route history, complex route editing, offline maps.

Consequence worth stating plainly: with no GPS smoothing, the drawn trail will show raw GPS noise. This is an accepted V1 trade-off, not a bug. **[Assumption]** that raw trails are good enough for a first prototype; we will judge this on real walks.

### Roadmap (planned, not committed)

| Version | Goal | Contents |
|---|---|---|
| V0 | Map experiment | Map renders, user taps points, points are displayed |
| V1 | Walking prototype | Tap points, generate route, start walk, GPS trail displayed |
| V1.1 | Route following | Planned route shown, distance from planned route, haptic off-route warning |
| V2 | Reliability | GPS filtering, background tracking, battery optimization, route saving |

## 3. Existing solutions

Findings from vendor pages and app-store listings, plus third-party articles where noted. We only record what the sources state; where a source is silent we say so.

| App | What it offers free | Behind paid tier | Gap relative to us |
|---|---|---|---|
| **Footpath Route Planner** | Route creation by tapping/drawing on the map with snap-to-roads-and-trails; distance measurement; keep up to 5 saved routes | Unlimited saved routes, turn-by-turn audio navigation, offline maps and routes, premium topographic maps and overlays | Closest comparable. Its free tier already covers planning with snapping. |
| **TouchTrails** | Draw routes on the map with a finger; GPX import and editing; distance/elevation; no account or login required | Snap to road, turn-by-turn navigation (including a warning when you leave the route), unlimited saved routes, GPX export, offline maps | Free tier does not snap to roads per the App Store listing, so snapping is a paid feature there. |
| **Komoot** | Route planning anywhere in the world; one free region for navigating/recording | Navigation and recording outside unlocked regions require purchasing regions or Premium; syncing routes to devices requires Premium for new users | Free navigation is limited to unlocked regions. |

**[Sourced]** details and links:

- Footpath pricing page lists snap-to-map route creation among its route-planning features and shows 5 saved routes for the free tier versus unlimited for Elite: <https://footpathapp.com/pricing/>
- Footpath App Store listing: free download with in-app purchases; "map routes with your finger and Footpath will snap to roads and trails"; keep up to 5 routes, unlimited with Elite; Elite adds turn-by-turn audio cues and premium offline maps; US prices listed as $3.99 monthly, $23.49 yearly, $1.99 single route pass: <https://apps.apple.com/KR/app/id634845718> (prices vary by region). A price-tracker snapshot dated 2026-05-07 shows the same US prices: <https://apppricinglab.com/iap/apple/634845718>
- Footpath Elite feature overview: <https://footpathapp.com/elite/>
- TouchTrails App Store listing (free with in-app purchases; Premium adds snap to road, turn-by-turn that warns when you leave the route, unlimited routes, GPX export, offline maps): <https://apps.apple.com/app/id6739418128>
- TouchTrails review stating no account or login is required at either tier and that route planning and GPX editing are free: <https://the5krunner.com/2026/07/07/touchtrails-route-planning-apple-watch-navigation/>
- Komoot plan structure (planning anywhere; navigating or recording needs an unlocked region; one free region offered): <https://support.komoot.com/hc/articles/10163258809626>
- Komoot device-sync change for new users (reported by BikeRadar): <https://www.bikeradar.com/news/new-komoot-users-send-to-devices>

**Corrections from the original draft of this note**

- The draft said TouchTrails restricts free users to 3 saved routes. **We could not find a source for that number.** The listing says Premium offers unlimited saved routes, but the free limit is not stated in the pages we found. Removed.
- The draft said Komoot requires an account and has a "steep learning curve" and pushes community features. We found no source for these. Removed. We only record the sourced region-unlock and Premium details.
- The draft claimed that "most mainstream fitness apps are reactive" and that existing planners are "often bloated". Neither is supported by evidence we collected. Removed.
- The draft said Footpath locks turn-by-turn navigation behind a subscription. This **is** supported (Elite).

**Not checked:** other apps (e.g., Plotaroute, Map My Walk, Strava, AllTrails) and the privacy practices of any competitor. We make no claim that competitors are less private.

## 4. How V1 works

1. The map is rendered by MapLibre. **[Sourced]** MapLibre has a Flutter plugin (`maplibre_gl`) that takes a style URL, local style, or raw style JSON, and a tile source is required: <https://maplibre.org/flutter-maplibre-gl/getting-started/>
2. The user's tapped points are sent to a routing engine to get a walkable route between them. For tapped waypoints this is a normal **route** request with a pedestrian profile, not "map matching".
   - Valhalla supports a `pedestrian` costing model for routing: <https://www.transit.land/documentation/routing-platform/valhalla>
   - The OSRM demo server documentation says it runs worldwide car, foot and bike profiles: <https://github.com/Project-OSRM/osrm-backend/wiki/Demo-server>
   - **Map matching** (Valhalla `trace_route` / `trace_attributes`, OSRM `match`) is for snapping a sequence of GPS-like points, such as a hand-drawn line or a recorded trail, to the road network. It is relevant if we later let users draw freehand, or snap recorded trails. Valhalla's map-matching docs: <https://valhalla.github.io/valhalla/api/map-matching/api-reference/>
3. The app uses the device GPS for the current position and draws the trail as the user walks. In V1 this happens with the app in use; background tracking is V2.

## 5. Services and usage policies (the main V1 risk)

All figures below are **[Sourced]** unless marked.

**Routing**

- The OSRM demo server is described as restricted to reasonable, non-commercial use, with no more than 1 request per second and no guarantees on uptime, latency or data updates: <https://github.com/Project-OSRM/osrm-backend/wiki/Demo-server>
- The OSRM API usage policy page (marked as applying to the previous demo server but "still good practice") asks for a valid identifying User-Agent, data-license attribution and states that excessive use will be blocked: <https://github.com/Project-OSRM/osrm-backend/wiki/Api-usage-policy>
- Valhalla's README says FOSSGIS hosts a public demo server under a fair-usage policy similar to the OSRM and Nominatim demo servers, somewhat enforced by rate limits: <https://github.com/valhalla/valhalla>
- A third-party API catalogue (not an official source) describes that Valhalla demo server as for development and testing, not production: <https://apis.io/plans/valhalla/open-source/>. Treat as **[To test]**: confirm against the FOSSGIS terms before shipping.
- Self-hosting Valhalla is possible via the project's Docker image (same README).

**Map tiles**

- The OpenStreetMap Foundation tile policy requires following its rules for `tile.openstreetmap.org`, says offline use is not permitted, and requires prior permission to distribute a heavy-usage app that uses those tiles: <https://operations.osmfoundation.org/policies/tiles/>
- MapLibre's own docs label its demo tiles as development-only and list other options (e.g., self-hosted PMTiles, commercial providers with limited free tiers): <https://maplibre.org/flutter-maplibre-gl/concepts/styles/>

**Implication:** a public release of V1 cannot simply point at free demo endpoints and assume it will keep working. Before release we need to pick tile and routing providers whose terms allow our usage, or self-host. This conflicts mildly with the "no backend of our own" goal, since self-hosting routing would mean running a server. **[Assumption]** that a paid or self-hosted option is acceptable if free options do not fit; this needs a decision.

**Privacy implication (important for how we describe the app):** the app has no backend of ours and no account, but tapped waypoints are sent to a third-party routing service, and map tiles requested reveal the area being viewed. The actual GPS trail can stay on the device by design **[Assumption]** (we must verify the app makes no other network calls). We should not describe the app as "fully on-device".

## 6. Technology choices

| Area | Current choice | Status |
|---|---|---|
| App framework | Flutter or React Native | **Undecided.** Flutter's `maplibre_gl` plugin exists (**[Sourced]**, link above). We have **not** verified the equivalent React Native library, so we cannot compare yet. |
| Map rendering | MapLibre | Chosen. Open-source, requires an external or self-hosted tile source. |
| Routing | Valhalla (pedestrian) or OSRM (foot) | **[To test]** which gives better walking routes for our test locations. |
| Persistence | None in V1 | Route history is a non-goal. Route saving is planned for V2. |
| Spatial math | Not needed in V1 | Distance-from-route checks arrive with V1.1. |

The earlier draft listed `SQLite`/`AsyncStorage`, `Turf.js` and background geolocation packages as part of V1. They are not needed for the V1 scope above and have been deferred to later versions.

## 7. Known difficulties by version

| Version | Difficulty | Evidence status |
|---|---|---|
| V1 | Free routing/tile services may not permit production use (section 5) | **[Sourced]** |
| V1 | Raw GPS trail will look noisy (no smoothing) | **[Assumption]**, to observe in testing |
| V1 | Walking-route quality on local paths (e.g., unmapped trails) | **[To test]** |
| V1.1 | Haptic warning while the phone is in a pocket or locked | **[To test]**. We have not verified what iOS and Android allow when the app is in the background. |
| V1.1 | Choosing a buffer width for "off route" | **[Assumption]**, to be tuned with real walks |
| V2 | Background location permissions and OS battery limits | **[To test]**. Not researched yet. |

## 8. Next steps (experiments, in order)

1. **V0 spike:** render a MapLibre map in the chosen framework with a development tile source; let the user tap points; display them.
2. **Routing test:** send 3 to 5 tapped waypoints from locations we actually walk to a Valhalla pedestrian route and an OSRM foot route; record whether the result follows real walkable paths, and note failures (parks, unmapped footpaths, private land).
3. **GPS test:** walk a short route with raw position updates and save the screenshots of the trail, so we know how noisy "no smoothing" really is.
4. **Provider decision:** based on steps 1 to 3 and the policies in section 5, choose tile and routing providers for a public release, and record the decision here.

## 9. Open questions

- Which platform(s) does V1 target first: Android, iOS or both?
- Which framework (Flutter or React Native), and on what criteria?
- Which tile provider and routing provider for release, and who pays or hosts?
- Is the project aimed at personal use, open-source learning, or a public release? This changes how much the usage-policy risk matters.
- What is the licence for the repository?

## 10. Change log

- **2026-10-02:** Rewrote the original research note to match the new V1 scope (tap points, no off-route alerts); added sources and claim labels; removed unsupported claims about competitors; moved off-route, smoothing, background tracking and saving to later versions.