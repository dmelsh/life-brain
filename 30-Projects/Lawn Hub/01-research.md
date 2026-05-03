---
type: research
project: "[[Lawn Hub]]"
created: 2026-05-02
updated: 2026-05-02
status: complete
tags: [lawn-hub, research, design]
---

# Lawn Hub — Research Doc

## Section 1 — App & Tool Analysis

### Irrigation Apps

#### Rachio
**What's built:** Full irrigation controller app. Zone list with run controls, weather-skip logic, flex daily scheduling, run history, and a "Yard Map" feature (Rachio 3+) that lets you draw zone boundaries over an aerial photo and place custom sprinkler head icons.

**Standout component:** Yard Map. You tap a zone in the list, then drop and drag polygon boundaries on a satellite image. Pin types are typed (rotor, spray, drip). Zone status shows real-time (running/skipped/upcoming). The app surfaces weather intelligence: "Skipped — 0.3" rain detected." Run history per zone is the second standout — a scrollable timeline of past runs with duration and estimated gallons.

**Tech stack:** Native iOS/Android. No public API documentation for the map layer, but zones are REST-accessible via the Rachio public API (v3). The Yard Map itself stores zone polygons server-side.

**What applies:**
- Zone-as-first-class-entity pattern — each zone has a name, icon, status, and history
- Color-coding active zones (blue pulse animation when running)
- Weather-skip surfacing directly in the zone card
- Per-zone run history timeline

**What doesn't:** Their map is a proprietary polygon editor (draw zone boundaries). For this hub, we're using simpler point pins, not polygon zone boundaries. Polygon editing is overkill — we just want a representative pin.

---

#### Hydrawise (Hunter)
**What's built:** Cloud-connected irrigation management. Zone cards with run-time controls, predictive watering (weather API integration), sensor inputs, and a zone map view using Google Maps satellite imagery.

**Standout component:** The "zone map" tab overlays colored pins on a live satellite tile map. Each pin is clickable to see zone detail. The color indicates zone status (green = watered today, yellow = due today, gray = skipped).

**Tech stack:** Native + web dashboard. Uses Google Maps for the satellite tile layer.

**What applies:**
- Color-coded zone health state at a glance
- Live tile satellite vs. static image — Hydrawise requires a live map dependency; our approach of a static screenshot is lighter and more reliable offline

**What doesn't:** Requires a live internet connection for tile loading. For a single property, a static satellite crop is faster to load and has no API dependency.

---

#### B-Hyve (Orbit)
**What's built:** Entry-level smart controller app. Simpler than Rachio/Hydrawise — zone list, basic scheduling, rain delay. No satellite map feature.

**Standout component:** Zone quick-run interface (tap zone, drag slider for duration, run). Immediate single-action control with no configuration required.

**What applies:** The quick-run slider UX for triggering an irrigation zone run directly from the hub's pin detail drawer.

**What doesn't:** No map, no data model — this is purely UX inspiration for the control action.

---

### Yard Mapping Apps

#### Husqvarna Automower App
**What's built:** Robot mower companion. After setup, the app shows the mower's real-time GPS position on a satellite image background. The mowed coverage area is rendered as a color overlay on the image as the mower traverses the yard. Also shows activity history per "zone" (defined mowing areas) and battery/charging status.

**Standout component:** Coverage overlay on satellite image. As the mower runs, a translucent fill tracks the mowed area. This is the closest analog in the wild to what we're building — a satellite image as the primary canvas with entity state overlaid on top.

**Tech stack:** Native mobile. GPS-based positioning.

**What applies:**
- Satellite image as primary canvas, not a tile map
- Entity state (mower position) as an overlay on the image
- Activity history per zone concept

**What doesn't:** GPS-accurate positioning. We're using manual pin placement (0–1 normalized coords), not GPS — appropriate for a fixed property.

---

#### iScape
**What's built:** AR and photo-based landscape design tool. You import a photo of your yard (or take one) and drag-and-drop plant/hardscape objects onto the image. Object library with 8,000+ plant species. Scale-aware placement.

**Standout component:** Photo-as-canvas drag-drop placement. You're literally placing objects on a photo of your yard. This is the exact UX metaphor we want — the property image is the map, and entities are placed on top of it.

**Tech stack:** Native iOS/Android. AR mode uses ARKit/ARCore.

**What applies:**
- Photo/image-as-canvas metaphor
- Typed entity placement (plant category → specific species → size)
- Before/after view toggle

**What doesn't:** AR and scale simulation are unnecessary complexity. We want a persistent data layer, not a design mockup tool.

---

#### Home Outside
**What's built:** Professional landscape planning tool, simplified for homeowners. Drag-and-drop plants and hardscape elements onto a site plan (usually uploaded image or drawn plan). Plant library with care requirements, bloom times, and zone hardiness.

**Standout component:** Bloom time visualization — a horizontal timeline showing which plants in your plan bloom in which months. Directly applicable to the Lawn Hub planting section.

**Tech stack:** Web-based. No mobile app.

**What applies:**
- Bloom time calendar per planting — a feature worth adding to Plant entity detail
- Plant hardiness zone (USDA zone 6b for Warson Woods) baked into the data model

**What doesn't:** Site plan mode is vector-based (drawn polygons). We're using a raster satellite image — simpler and more visually grounded.

---

### Plant Log Apps

#### Gardenize
**What's built:** Garden journal app. Create individual plant profiles with photo history, care log (watering, fertilizing, pruning, pest treatment), seasonal notes, and reminders. Organized by garden areas.

**Standout component:** Timeline/activity log per plant. Every action (watered, fertilized, treated) gets a dated entry with optional photo. You can scroll back through a plant's entire history. This is the core UX for our Observation and Treatment entities.

**Tech stack:** Native iOS/Android. Cloud sync. No API.

**What applies:**
- Per-entity activity timeline (date → action → notes → photo)
- Garden area grouping (maps to our named area taxonomy)
- Care reminder system

**What doesn't:** Per-plant reminders are too granular for a lawn hub. We're managing zones and beds, not individual specimens.

---

#### Planta
**What's built:** Plant care companion. AI-based species identification, personalized watering schedules, light/humidity tracking, and push notifications. Strong visual design.

**Standout component:** "Health score" per plant — a composite metric (watering regularity, light exposure, recent issues) displayed as a circular progress ring. Clean card-based plant list.

**Tech stack:** Native iOS/Android. Subscription model.

**What applies:**
- Health/status summary per entity — a green/yellow/red composite state badge on each pin
- Notification cadence for care actions (maps to treatment schedule reminders)

**What doesn't:** AI species identification. We know what we planted. Planta's watering schedule logic is also too micromanaged for outdoor zones — Rachio handles that.

---

#### From Seed to Spoon
**What's built:** Vegetable gardening app. Companion planting, days-to-harvest countdown, planting calendar by USDA zone.

**Standout component:** USDA zone-aware planting calendar. Enter your zip code → get a personalized "plant by this date" calendar for each crop. The calendar is color-coded by season window (start indoors / transplant outside / harvest window).

**Tech stack:** Native iOS/Android.

**What applies:**
- USDA zone awareness for timing (Z6b for Warson Woods)
- Calendar integration for bloom/harvest timing (applicable to planting entities)

**What doesn't:** Vegetable-specific (no lawn care, no pest tracking, no irrigation). Narrow use case.

---

### Obsidian Garden Journal Templates

Community patterns worth modeling:

1. **Per-plant note with YAML frontmatter** (Obsidian Forum, 2022): Most users settle on one note per plant with frontmatter fields: `plant_name`, `species`, `location`, `planted_date`, `zone`, `status` (active/removed/dormant), and a `## Log` section at the bottom with dated bullet entries. Exactly the schema we should adopt for Plant entities.

2. **Area-based note with embedded plant list** (Obsidian Forum plant tracking thread, April 2024): Group plants by bed/area as a list in the area note, with wikilinks to individual plant notes. Mirrors our named-area taxonomy exactly — `[[front-yard-garden]]` links to a page listing `[[Lily - Starfighter]]`, `[[Lily - Vestaro]]`, etc.

3. **Periodic garden log (daily/weekly notes)**: Using the Periodic Notes plugin, some users maintain a `garden` section in daily notes for quick observations, then distill to plant notes weekly. Maps well to our Observation entity — `life-brain/Lawn Hub/observations/YYYY-MM-DD.md`.

---

### Coordinate Overlay Systems

#### Frigate NVR Zone Editor
**What's built:** Motion detection zone editor for IP cameras. You draw polygon zones on a live camera feed. Coordinates are stored as normalized 0–1 values (`[x,y]` pairs) relative to the image frame.

**Standout component:** The coordinate system is exactly what we want — `[0.033, 0.306]` means "3.3% from the left, 30.6% from the top" of the image. Image-agnostic and resolution-independent. If you swap the base image (e.g., updated satellite crop), the coordinates remain geometrically valid as long as the framing is consistent.

**What applies:** The normalized 0–1 coordinate system. This is the definitive model to adopt. Frigate has proven this works in production at scale.

---

#### UniFi Protect Zone Editing
**What's built:** Similar to Frigate — polygon zone drawing over a live video feed. Zones are used for motion alerts. Coordinates stored as pixel fractions.

**Standout component:** Multi-zone overlay with color-coded polygon fills (each detection zone gets a distinct color). Zone states (armed/disarmed) visualized directly on the image with opacity.

**What applies:**
- Color coding zones by type (armed/disarmed maps to zone enabled/disabled)
- Opacity overlay for zone state visualization

---

#### HA Lovelace `picture-elements` Card

**What it is:** A Home Assistant dashboard card that places entity icons, state badges, and service buttons at absolute `left`/`top` percentage positions on top of any image.

**Standout community examples:**

1. **r/homeassistant — Interactive Floorplan showcase** (community.home-assistant.io "Show off your picture-elements" thread, 2018–ongoing, 1000+ replies): Users have placed sprinkler zone icons on satellite images or hand-drawn site maps. Icon color changes dynamically based on entity state (blue = running, gray = idle). This is the most-cited HA approach for an image-as-map UI.

2. **r/homeassistant — "GUI for a dashboard that shows sprinklers?" thread** (May 2024): User explicitly asked for picture-elements over a yard satellite image to visualize which zone is running. Top answer used `state-icon` elements at percentage positions with CSS color overrides. Community confirmed this approach works and shared YAML snippets.

3. **r/homeassistant — Custom irrigation dashboard** (July 2025, 128 upvotes): User built a full HA dashboard showing irrigation zones on a property map with real-time state, history, and controls — all using picture-elements + custom CSS. Post includes screenshots showing exactly the UX pattern we're targeting.

4. **r/smarthome — Zircon3D** (May 2025): 3D floorplan tool for HA. Shows the community appetite for richer spatial visualizations. While overkill for this hub, the engagement (~117 upvotes) confirms user demand for spatial property visualization.

**What applies directly:** The `left`/`top` percentage system in picture-elements is the same normalized coordinate concept. Our `pins.json` with `x: 0.52, y: 0.68` maps directly to `left: 52%; top: 68%` in CSS — the mental model is identical.

---

### Pest/Lawn Log Patterns

#### TruGreen App (customer portal)
**What's built:** Customer-facing service tracking. After each visit, you get a notification with: service performed, products applied (with EPA registration numbers), coverage area, next scheduled visit. History view shows all past applications with dates.

**Standout component:** Per-visit application record with product list and areas covered. This is the exact schema for our WeedTreatment and PestTreatment entities.

**What applies:**
- Application log: date, product, mix rate, area covered, technician notes
- Next-scheduled display on the entity card

**What doesn't:** Cloud-managed schedule. We're self-managed — no service provider to push visit records.

---

## Section 2 — Reddit/YouTube Sweep (Past 12 Months)

**Source 1 — r/homeassistant: "GUI for a dashboard that shows sprinklers?" (May 2024)**
URL: reddit.com/r/homeassistant/comments/1cw0u4v/gui_for_a_dashboard_that_shows_sprinklers/
Summary: User asked for a visual zone map in HA. Community response: use picture-elements card with zone icon states over a property photo. Multiple users shared working YAML. Confirms picture-elements as the go-to HA approach for this pattern. Also discussion of Mushroom Cards + custom:button-card for zone control tiles.

**Source 2 — r/Irrigation: "Irrigation Mapping over Satellite/Aerial Imagery" (August 2023)**
URL: reddit.com/r/Irrigation/comments/163pv84/irrigation_mapping_over_satelliteaerial_imagery/
Summary: A developer shared a web app they built for mapping irrigation systems over satellite imagery. Community response was positive — many users expressed interest in something beyond Rachio's built-in Yard Map. Key insight: users want GPS-optional mapping — they don't care about real-world coordinates, they want their property image with dots on it.

**Source 3 — r/homeassistant: "Custom irrigation system and dashboard in HA" (July 2025)**
URL: reddit.com/r/homeassistant/comments/1mav0hm/custom_irrigation_system_and_dashbaord_in_ha/
Summary: 128 upvotes, 46 comments. Full HA irrigation dashboard with per-zone cards, run history, and visual zone map. Shows exactly the appetite for this kind of hub and validates the technical approach (HA integration + React frontend for richer features than HA can provide natively).

**Source 4 — r/selfhosted: HortusFox (self-hosted plant management)**
URL: github.com/danielbrendel/hortusfox-web
Summary: Open source self-hosted plant management app. Shows the community appetite for local-first, self-hosted garden tracking. HortusFox is PHP-based — not relevant technically, but confirms the market: people want a local alternative to Gardenize/Planta.

**Source 5 — r/selfhosted/r/homeassistant: Plant-It (open source Android app)**
URL: github.com/MDeLuise/plant-it
Summary: Open source Android + REST API plant journal. Has a proper API (plants, events, reminders) and supports self-hosting via Docker. More relevant technically — their event schema (plant_id, type, date, notes) is directly applicable to our Treatment and Observation entities.

**Source 6 — YouTube: HA floorplan tutorials (2024–2025)**
Key video type: "Home Assistant picture-elements floorplan" tutorials. The Strawberry Sec tutorial (January 2025) specifically covers creating pixel-art property maps and wiring entity icons to coordinates — directly applicable technique. The tutorial pattern is: (1) prep image, (2) identify pixel coordinates as percentages, (3) place state-icon elements at those coordinates, (4) add tap actions.

---

## Section 3 — Design Standards

### Apple HIG Map Patterns

**FAB (Floating Action Button):** Apple HIG positions primary actions as prominent buttons that float above map content. For the Lawn Hub, a FAB in the bottom-right corner labeled "＋ Add Pin" or "Log Observation" keeps the primary action immediately accessible without obscuring the map.

**Map interaction patterns:**
- Tap a pin → trigger a detail panel (not navigate away from the map)
- Long-press → context menu (edit / delete / log action)
- Pinch-to-zoom on the satellite image (if we implement zoom)
- Map should scroll/pan independently from the page chrome

**Sheet vs. Modal:** Apple HIG recommends sheets (bottom drawer) for map pin details on mobile — the map remains partially visible behind the sheet, maintaining spatial context. Full-screen modals break the map → detail → back → map mental model.

---

### Material 3 Layered Surface Hierarchy

M3 defines surface layers by elevation (tonal fills for light mode, elevation overlays for dark mode). For this hub:
- **Layer 0 (Base):** Satellite image — the ground truth
- **Layer 1 (Pins):** Icon markers floating above the image — use `shadow-md` from Tailwind
- **Layer 2 (Active state):** Selected pin → ring highlight + popover tooltip above the pin
- **Layer 3 (Detail panel):** Bottom drawer slides up, sits above the map — `bg-card` with `shadow-xl`

---

### Mapbox/Leaflet Patterns for Image-as-Map

**Image overlay approach:** Leaflet's `L.imageOverlay(url, bounds)` lets you treat any raster image as a map canvas with LatLng-based coordinates. However, for a single-property satellite crop, this is overkill — you get the full Leaflet dependency for no benefit. The simpler pattern is a positioned `<div>` with `position: relative` containing the image, and pin markers as absolutely-positioned children at `left: x*100%; top: y*100%`.

**Clickable pin with detail drawer (Leaflet pattern):**
```
Pin click → L.popup() attached to marker OR
Pin click → Custom React state (selectedPin) → Bottom drawer opens
```
For this hub, the React state approach (no Leaflet dependency) is preferred. The image is static, not a tile map.

**shadcn/ui primitives — which fits which use case:**

| Component | Use Case in Lawn Hub | Why |
|-----------|---------------------|-----|
| **Sheet** (side panel) | Pin detail on desktop — slides from the right, keeps map visible on left | Sheet is modal-like with full height; good for rich entity detail (history, actions) on wider screens |
| **Drawer** (bottom panel) | Pin detail on mobile — slides from bottom, feels native to iOS/Android | Drawer uses Vaul under the hood; swipe-to-dismiss works perfectly for a map detail interaction |
| **Popover** | Quick status tooltip above a pin | Show zone status, last-watered date, or plant health on hover/tap before committing to full detail |
| **Tooltip** | Pin label on hover | Show entity name on hover without opening the full panel |

**Recommendation:** Use `Drawer` for pin detail on mobile (primary target), `Sheet` (right side) for desktop. Use `Popover` for quick-peek status preview on hover. Use `Tooltip` sparingly — only for icon-only pins where the label isn't visible.

---

### Image-as-Map vs. Geo Coordinates — Explicit Recommendation

**Recommendation: Image-as-map with normalized 0–1 (x, y) pixel coordinates.**

**Justification:**

1. **Single property scope:** There's exactly one property. The entire value of geo coordinates (WGS84 lat/lon) is projecting multiple locations onto a consistent coordinate system. With one satellite crop at fixed zoom/framing, normalized pixel coordinates are equally stable and far simpler.

2. **No mapping library needed:** Geo coordinates require a map projection library (Leaflet, Mapbox, etc.) to convert lat/lon to screen pixels. Normalized coordinates can be applied directly as CSS: `left: ${x * 100}%; top: ${y * 100}%`. Zero dependencies.

3. **Image stability:** The satellite image was captured at a known zoom level and framing (ESRI z20, 512×512). As long as the image is re-cropped to the same framing, coordinates remain valid. Registration anchor points (e.g., corners of the house footprint, fixed hardscape features) let you verify alignment if the image is ever re-fetched.

4. **Proven in production:** Frigate NVR uses this exact system for camera zone polygons at scale. UniFi Protect uses it too. The HA picture-elements card uses it (as CSS percentages). It's the standard pattern for image-overlay UIs.

5. **Geo coordinates add complexity with no benefit here:** The property isn't moving. You're not comparing coordinates across multiple properties. You don't need routing or distance calculations. The overhead of WGS84 precision (6 decimal places, requires CRS understanding) isn't justified.

**The only scenario where geo coordinates win:** if you later want to integrate live weather radar overlays or compare your property to a tile map layer. A migration path exists (anchor point lat/lon lookup → transform matrix) but isn't needed now.

---

### Color & Icon Palette

**Color palette by entity category:**

| Category | Primary Color | Hex | Tailwind Class | Why |
|----------|--------------|-----|----------------|-----|
| Irrigation Zone | Blue | `#3B82F6` | `text-blue-500` | Universal water = blue association |
| Plant/Planting | Green | `#22C55E` | `text-green-500` | Life, growing things |
| Weed Treatment | Teal | `#14B8A6` | `text-teal-500` | Chemical/lawn care (distinct from plants) |
| Pest Treatment | Orange-Red | `#EF4444` / `#F97316` | `text-red-500` / `text-orange-500` | Danger/caution = red-orange |
| Light Fixture | Amber | `#F59E0B` | `text-amber-500` | Light/warmth |
| Audio Zone | Purple | `#A855F7` | `text-purple-500` | Conventional audio/media color |
| Observation | Gray | `#6B7280` | `text-gray-500` | Neutral log/note |
| Active State | Bright Cyan | `#06B6D4` | `text-cyan-400` | "Something is happening now" pulse |

**State modifiers:**
- Active/running: add `animate-pulse` ring in the active color
- Disabled/inactive: reduce opacity to `opacity-40`
- Issue/alert: overlay a small badge with `text-red-500 bg-red-100`

**Icon library (Lucide — already in ACC stack):**

| Entity | Lucide Icon | Notes |
|--------|-------------|-------|
| Irrigation Zone | `Droplets` | Multi-drop = irrigation |
| Plant/Planting | `Flower2` or `Sprout` | Use Sprout for new plantings, Flower2 for established |
| Weed Treatment | `Syringe` or `FlaskConical` | Chemical application |
| Pest Treatment | `Bug` | Explicit and immediately recognizable |
| Light Fixture | `Lamp` or `Lightbulb` | Use Lamp for outdoor fixtures |
| Audio Zone | `Volume2` or `Speaker` | |
| Observation | `StickyNote` or `Eye` | Use Eye for "I noticed something" |
| Active/Running | `Zap` badge overlay | Small badge on the pin |

---

## Section 4 — Design Thesis

Good outdoor property dashboards have failed homeowners by pulling in opposite directions: either they're too abstract (a list of zone names and run durations, completely disconnected from spatial reality) or they try to be full GIS applications (tile map layers, WGS84 coordinates, polygon editors), which impose engineering overhead disproportionate to managing one backyard. The right answer for a single-homeowner outdoor hub is a satellite image of the actual property — the one thing every owner has a mental model of already — treated as a first-class canvas, with typed entities (zones, plants, treatments, observations) pinned to it as interactive markers. State flows one direction: live data (Rachio zone status, HA lights) overlays the static spatial context without moving complexity to the image layer itself. The interaction model should be gesture-native: tap a pin to get a bottom drawer with full entity context, long-press to act, swipe to dismiss. The hub earns trust through completeness — when something happened in the yard (a weed treatment, a rain event, a pest observation), it's logged here, and the property image makes the log spatial and memorable rather than just a timestamped list. This is what good looks like: a satellite image you recognize, dots in the right colors, and a rich but uncluttered detail panel one tap away.

---

*Research compiled: 2026-05-02. Sources: Rachio support docs, r/homeassistant, r/Irrigation, r/selfhosted, HA community forums, GitHub (HortusFox, Plant-It, Frigate docs), Apple HIG, shadcn/ui docs, Lucide icons. Web research current as of May 2026.*
