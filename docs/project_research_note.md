# Project Idea
*   **What the idea is:** A mobile app allowing users to map out their desired walking route *before* they start walking, and then tracking their real-time progress along that specific route using GPS.
*   **What problem it solves:** Most mainstream fitness apps are reactive (tracking where you went *after* the fact) rather than proactive (planning where you want to go). Existing proactive planners are often bloated, feature-heavy, or require paid subscriptions.
*   **What the intended user experience is:** Frictionless and immediate. A "Zero-Tap Launch" where the user opens the app without logging in, draws a route with their finger, pockets the phone, and walks. It acts as a "zen walker" tool with no complex metrics, relying on haptic feedback (vibrations) to keep the user on track.
*   **What the end goal of the project is:** To deliver a privacy-first, purely on-device walking companion that strictly serves the utility of custom route planning and tracking without the noise of social networks or intense fitness metrics.

# How It Works
1.  The user opens a map interface and drags their finger to draw a rough path.
2.  The app queries a routing engine to perform "map matching," which snaps the sloppy drawing to real-world walkable paths and roads.
3.  The user starts the walk; the app locks onto their GPS location.
4.  As the user walks, a background service constantly updates their location, using spatial math to check if they are still inside an "invisible tube" (buffer zone) around the planned route. 
5.  If they wander outside the tube, the app triggers a haptic alert.

# Existing Solutions
*   **Footpath Route Planner:** 
    *   *What it does:* Allows users to trace a route and snaps it to trails/roads.
    *   *Relation/Gap:* Closest competitor. However, many of its core turn-by-turn navigation features are locked behind a premium subscription [Researched]. Our app aims for frictionless, free basic utility.
*   **TouchTrails:** 
    *   *What it does:* Traces routes on a map to calculate distance and track elevation.
    *   *Relation/Gap:* Very similar utility, but heavily restricts users (e.g., only 3 saved routes in the free tier). 
*   **Komoot:** 
    *   *What it does:* Advanced trail planner for hikers/bikers with surface types, waypoints, and community recommendations.
    *   *Relation/Gap:* Highly complex with a steep learning curve. Requires an account and pushes community features. Leaves a gap for a much simpler, lightweight alternative.

# Proposed Tech Stack
*   **Frontend:** `Flutter` or `React Native`
    *   *Why:* Cross-platform capabilities. You can build iOS and Android versions simultaneously without writing Swift and Kotlin separately.
*   **Backend:** None (Serverless / Local)
    *   *Why:* Keeps the V1 simple, highly private, and zero-cost to host.
*   **Database:** Local Storage (`SQLite`, `AsyncStorage`, or `SharedPreferences`) [Reasoned]
    *   *Why:* To locally save previously drawn routes without needing a cloud database or user accounts.
*   **APIs / Services:**
    *   **Map Renderer:** `MapLibre GL` (Free, open-source alternative to expensive Google Maps/Mapbox APIs).
    *   **Routing Engine:** `Valhalla` or `OSRM` (Public/free tier instances to handle the "Map Matching" of turning screen taps into real road paths).
*   **AI/ML:** Not necessary for MVP. [Reasoned]
*   **Other Important Tools:**
    *   `Turf.js` (or Dart equivalent): Industry standard for client-side spatial math (calculating distance, checking if a GPS coordinate is inside the route's buffer zone).
    *   `react-native-geolocation-service` (or Flutter `geolocator`): Essential for capturing location data.

# Main Technical Difficulties
*   **Map Matching:** Raw finger swipes are just screen coordinates. Translating these to legal walking paths requires passing the coordinates through a routing engine (OSRM/Valhalla) so the line doesn't cut through buildings.
*   **GPS Drift and Jitter:** GPS signals bounce and degrade under trees or buildings, causing the user's location dot to jump erratically. You will need a smoothing algorithm to ignore wild jumps and accurately calculate distance.
*   **Background OS Kills:** iOS and Android aggressively kill apps that use GPS in the background to save battery. Implementing correct background permissions and background-task handlers is historically the biggest hurdle in fitness apps.
*   **Off-Route Detection:** You must code a continuous calculation checking if the user's smoothed GPS dot remains within a defined polygon (the "invisible tube" buffer) around the route, accommodating standard GPS inaccuracy.

# Current Direction
*   **What we should build:** A minimal, lightweight, privacy-first "zen" walking route planner. 
*   **What the MVP should contain:** An interactive map to draw a route, a map-matching API to snap the route to real paths, background GPS tracking, and a simple off-route haptic alert. No logins, no backend.
*   **What we should avoid overcomplicating:** Do not build a backend. Do not add social features, calorie counters, speed leaderboards, or user accounts.
*   **What the next step should be:** Create a basic React Native/Flutter sandbox with `MapLibre` and test out sending a string of coordinates to a free `OSRM` or `Valhalla` instance to see how well it snaps to local roads. Prove the map-matching concept first.