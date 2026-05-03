---
type: model
project: "[[Lawn Hub]]"
created: 2026-05-02
updated: 2026-05-02
status: complete
tags: [lawn-hub, data-model]
---

# Lawn Hub — Data Model

## Section 1 — Coordinate System

### Recommendation: Normalized 0–1 (x, y) over the Satellite PNG

Store all entity positions as normalized float coordinates `(x, y)` where:
- `x = 0.0` = left edge of the image, `x = 1.0` = right edge
- `y = 0.0` = top edge of the image, `y = 1.0` = bottom edge

**To render at pixel position:** `pixel_x = x * image_width`, `pixel_y = y * image_height`
**To render as CSS:** `left: ${(x * 100).toFixed(2)}%; top: ${(y * 100).toFixed(2)}%`

**Base image spec:**
- Primary: `warson-property-satellite.jpg` — 512×512 px, ESRI z20, centered on lat=38.6061, lon=-90.3823
- Wide: `warson-property-wide.jpg` — 768×768 px (use for full-property context if needed)

---

### Updating the Base Image Without Losing Pin Positions

**Problem:** If you re-fetch the satellite image at a different zoom, pan, or resolution, all existing pin coordinates will be spatially wrong.

**Solution: Named anchor points (registration marks)**

Maintain 3–4 named anchor points in `pins.json` that correspond to clearly identifiable, permanent features on the property (corners of the house footprint, the patio slab corner, the front driveway edge). When re-fetching the base image:

1. Re-fetch the image at the **same zoom level and center coordinates** (ESRI z20, lat=38.6061, lon=-90.3823)
2. Verify anchor point positions by checking that anchor pins still visually land on the correct features
3. If the image has drifted (different center crop), compute a simple translation offset: `new_x = old_x + dx`, `new_y = old_y + dy` using the delta from one anchor point
4. If scale changed, compute a scale factor from two anchor points and apply uniformly

**Anchor point approach for `pins.json`:**
```json
{
  "anchors": [
    { "id": "anchor-nw-corner", "label": "NW corner of house", "x": 0.38, "y": 0.22 },
    { "id": "anchor-patio-sw", "label": "SW corner of patio slab", "x": 0.55, "y": 0.72 },
    { "id": "anchor-driveway-end", "label": "End of front driveway", "x": 0.30, "y": 0.08 }
  ]
}
```

If all three anchors still land correctly on a new image, all other pin coordinates are valid. If they've drifted, you have a transform matrix to fix them.

**In practice:** ESRI z20 satellite imagery at the same center coordinates re-fetches within 1–3 pixels at this scale — anchor verification will rarely need intervention.

---

### Why Not Geo Coordinates?

Geo coordinates (WGS84 lat/lon) are the right choice when:
- You have multiple properties or need to compare locations
- You want to integrate tile map layers (street view, terrain)
- You need real-world distance calculations (pipe lengths, zone coverage radius)
- You want GPS-aware mobile features (walking to a pin triggers actions)

None of these apply here. The property is fixed. The image is a crop at known zoom. Normalized pixel coordinates are:
- Simpler to implement (pure CSS, no projection library)
- Easier to debug (0.5, 0.5 is the center — visually obvious)
- Proven in production (Frigate NVR, UniFi Protect, HA picture-elements)
- Stable against re-fetches as long as framing is preserved

Migration path if you ever need geo: identify 3 anchor points, look up their lat/lon from the original API call (ESRI tile math is deterministic), and compute a pixel→WGS84 transform matrix. That's a future problem.

---

## Section 2 — Entity Schemas

All entity schemas below define the data structure. Storage location is noted per entity.

---

### Plant / Planting
**Stored in:** `life-brain/Lawn Hub/plantings/SLUG.md` (markdown frontmatter)

**Fields:**
```yaml
---
type: planting
id: planting-z7-lily-starfighter      # slug, unique
name: "Lily — Starfighter"            # display name
species: "Lilium 'Starfighter'"       # botanical name (optional)
category: lily                         # lily | peony | hollyhock | daylily | annual | perennial | shrub | groundcover
area: z7-backyard-front               # named area ref (see Section 4)
location:                             # pin coordinates on satellite image
  x: 0.52
  y: 0.72
usda_zone: "6b"                       # Warson Woods MO hardiness zone
planted_date: 2026-04-19              # ISO date
status: established                   # seedling | established | dormant | removed
sun: full                             # full | partial | shade
bloom_start: 2026-07-01               # expected bloom start (ISO date, approximate)
bloom_end: 2026-08-15                 # expected bloom end
watering_zone: zone-7                 # links to irrigation zone
notes: "Costco bulk purchase Apr 2026, planted with Jay's mulch job"
tags: [planting, retaining-wall, sun-garden]
---

## Log

- 2026-04-19 — Planted along south retaining wall bed, ~18" spacing, amended soil
- 2026-05-01 — Mulched (Jay's Mulch, $3,400 job). Looking healthy.
```

**Concrete example:** See above (Lily Starfighter in Z7 bed)

---

### Irrigation Zone
**Stored:** Pulled live from Rachio API. Hub stores only coordinate + display name in `pins.json`.

**Rachio API fields used (live pull):**
- `id` — Rachio zone ID (integer, 1–8)
- `name` — Zone name (string)
- `enabled` — boolean
- `runtime` — current run state from Rachio webhook/poll
- `lastWateredDate` — ISO timestamp
- `nextRunDate` — ISO timestamp (from schedule)
- `runtimeHistory[]` — array of past runs (date, duration_seconds)

**Hub-side schema (pins.json entry only):**
```json
{
  "id": "zone-7-backyard-front",
  "type": "irrigation_zone",
  "label": "Backyard Front (Z7)",
  "x": 0.52,
  "y": 0.68,
  "entity_ref": "rachio:zone:7"
}
```

**State display:** Real-time zone state is fetched on page load + polled every 60s (or on Rachio webhook if HA integration is wired). No duplication of Rachio's zone data — the hub is a read layer, not a write store for irrigation data.

**Concrete example (Rachio zone 7):**
- Name: "Backyard Front"
- Enabled: true
- Last watered: 2026-05-01
- Next run: per Rachio flex schedule
- Covers: Z7 sun garden beds (peonies, daylilies, lilies), drip line + overhead

---

### Rain Event
**Stored:** Pulled live from HA weather entity. No duplication storage.

**HA entity:** `weather.home` (or `sensor.warson_woods_precipitation` if configured)
**Rachio also surfaces weather-skip events** — these are pulled from the Rachio schedule history when `type == "WEATHER_SKIP"`.

**Display in hub:** Rain events appear as a timeline entry in the zone detail drawer (pulled from Rachio history) and as a global "last rain" badge on the map toolbar.

**No separate storage needed.** All rain data lives in Rachio history + HA weather entity.

---

### Weed Treatment
**Stored in:** `life-brain/Lawn Hub/treatments/weed/YYYY-MM-DD-SLUG.md`

**Fields:**
```yaml
---
type: weed_treatment
id: weed-2026-05-01-tenacity-pre
date: 2026-05-01                      # ISO date of application
product: "Tenacity (Mesotrione)"      # product name
mix_rate: "2 oz + 0.5 oz NIS per gallon"
area: full-lawn                       # named area or "full-lawn"
area_sqft: 15000                      # coverage area in sq ft
coverage_method: backpack-sprayer     # backpack-sprayer | hose-end | granular | broadcast
target: crabgrass-pre-emergent        # what you were treating
volume_applied: "~12 gallons (4x 3-gal tanks)"
notes: "Used Hi-Yield Spreader Sticker + Turf Mark. Full coverage. Whitening expected in 2-3 weeks."
next_application: 2026-06-01          # expected next round (6-8 week window)
location:                             # pin on map (optional for full-lawn; use for spot treatments)
  x: null
  y: null
tags: [weed, tenacity, pre-emergent, crabgrass]
---
```

**Concrete example:** See above (Tenacity pre-emergent, 2026-05-01, full lawn, 15,000 sq ft)

---

### Pest Treatment
**Stored in:** `life-brain/Lawn Hub/treatments/pest/YYYY-MM-DD-SLUG.md`

**Fields:**
```yaml
---
type: pest_treatment
id: pest-2026-05-01-onslaught-indoor
date: 2026-05-01
product: "Onslaught FastCap Spider & Scorpion"
mix_rate: "0.5 oz per gallon"
target: brown-recluse                 # brown-recluse | ants | mosquitoes | stink-bugs | gnats
area: indoor-full                     # indoor-full | perimeter | backyard | specific named area
notes: "Round 1 of 4 annual. Full interior: baseboards, closets, basement perimeter, garage. Next Round 2 due late June/July."
next_application: 2026-07-01
source: diy                           # diy | pestie | contractor
location:
  x: null                             # null = non-spatial (full interior/exterior)
  y: null
tags: [pest, brown-recluse, onslaught]
---
```

**Concrete example:** See above (Onslaught FastCap Round 1, indoor, 2026-05-01)

---

### Light Fixture
**Stored:** Pulled live from HA `light.*` entities. Hub stores coord + display name in `pins.json` only.

**HA entity fields used:**
- `entity_id` — e.g., `light.back_patio_string_lights`
- `state` — `on` / `off`
- `brightness` — 0–255
- `attributes.friendly_name` — display label
- `last_changed` — last state change timestamp

**Hub-side schema (pins.json entry):**
```json
{
  "id": "light-back-patio",
  "type": "light_fixture",
  "label": "Back Patio Lights",
  "x": 0.60,
  "y": 0.75,
  "entity_ref": "ha:light.back_patio_string_lights"
}
```

**Actions from detail drawer:**
- Toggle on/off (POST to HA REST API `/api/services/light/turn_on` / `turn_off`)
- Set brightness (slider)

---

### Audio Zone
**Stored:** Pulled live from HA `media_player.*`. Hub stores coord + display name in `pins.json` only.

**Hub-side schema (pins.json entry):**
```json
{
  "id": "audio-backyard",
  "type": "audio_zone",
  "label": "Backyard Speaker",
  "x": 0.65,
  "y": 0.60,
  "entity_ref": "ha:media_player.backyard_speaker"
}
```

**HA entity fields used:**
- `state` — `playing` / `idle` / `off`
- `attributes.media_title` — current track
- `attributes.volume_level` — 0.0–1.0

**Actions from detail drawer:**
- Volume slider (PUT to HA REST)
- Play/pause

---

### Observation
**Stored in:** `life-brain/Lawn Hub/observations/YYYY-MM-DD.md`
One file per day; multiple observations can appear in the same daily file under separate headings.

**Fields:**
```yaml
---
type: observation
date: 2026-05-01
tags: [observation, lawn]
---

## [2026-05-01 14:30] Front Yard Hill — Tenacity Whitening

- **Area:** front-yard-hill
- **Category:** lawn-health
- **x:** 0.28
- **y:** 0.35
- **Observation:** Visible blanching starting ~3 days after Tenacity application. Normal response. Should green up in 2-3 weeks.
- **Action taken:** None — expected response.
- **Follow-up:** Check 2026-05-15 for recovery.

---

## [2026-05-01 15:00] Backyard Left — Clover Spot

- **Area:** backyard-left
- **Category:** weed
- **x:** 0.40
- **y:** 0.58
- **Observation:** Two clover patches near fence line, ~2 sq ft each.
- **Action taken:** Flagged for spot treatment with Tenacity post-emergent next week.
- **Follow-up:** Log treatment when applied.
```

**Why daily files instead of per-observation files:** Observations are ephemeral — they're journal entries, not entities. Daily files are consistent with the life-brain daily notes pattern and easy to grep. The `x`/`y` fields are optional metadata for map rendering.

---

### Action
**Definition only — not stored as files; defined as code.**

An Action represents a command that can be executed against an external service (HA, Rachio) or an OpenClaw cron job. Actions appear in entity detail drawers as buttons.

**Action schema (TypeScript interface):**
```typescript
interface Action {
  id: string;                    // unique action identifier
  label: string;                 // display label ("Run Zone", "Toggle Light")
  icon: string;                  // Lucide icon name
  entityRef: string;             // e.g., "rachio:zone:7" or "ha:light.back_patio"
  serviceCall: {
    provider: "rachio" | "ha";   // which API to call
    endpoint: string;            // e.g., "/api/services/light/turn_on"
    method: "GET" | "POST" | "PUT";
    body?: Record<string, unknown>;
  };
  confirmRequired: boolean;      // show confirmation dialog before executing
  feedbackType: "toast" | "badge_update"; // how to show success
}
```

**Concrete examples:**
```
Action: Run Zone Now (Zone 7, 5 min)
  provider: rachio
  endpoint: /zones/{id}/start
  body: { duration: 300 }
  confirmRequired: false

Action: Toggle Patio Lights
  provider: ha
  endpoint: /api/services/light/toggle
  body: { entity_id: "light.back_patio_string_lights" }
  confirmRequired: false

Action: Log Weed Treatment
  → opens a form sheet to create a new weed treatment file in life-brain
  → no external API call; writes to Obsidian via filesystem
```

---

## Section 3 — `pins.json` Schema

`pins.json` is the single source of truth for **spatial position** of all entities on the satellite image. It is the bridge between the static image and the entity data in life-brain, Rachio, or HA.

**File location:** `~/projects/acc/frontend/public/pins.json` (or served via FastAPI static endpoint)

**Full schema:**

```json
{
  "version": 1,
  "base_image": "warson-property-satellite.jpg",
  "image_size": [512, 512],
  "anchors": [
    {
      "id": "anchor-nw-corner",
      "label": "NW corner of house roof",
      "x": 0.38,
      "y": 0.22
    },
    {
      "id": "anchor-patio-sw",
      "label": "SW corner of patio slab",
      "x": 0.55,
      "y": 0.72
    },
    {
      "id": "anchor-driveway-end",
      "label": "Front driveway street end",
      "x": 0.30,
      "y": 0.08
    }
  ],
  "pins": [
    {
      "id": "zone-1-front-street",
      "type": "irrigation_zone",
      "label": "Front Yard Street (Z1)",
      "x": 0.30,
      "y": 0.15,
      "entity_ref": "rachio:zone:1",
      "visible": true,
      "group": "irrigation"
    },
    {
      "id": "zone-7-backyard-front",
      "type": "irrigation_zone",
      "label": "Backyard Front (Z7)",
      "x": 0.52,
      "y": 0.68,
      "entity_ref": "rachio:zone:7",
      "visible": true,
      "group": "irrigation"
    },
    {
      "id": "planting-z7-lily-starfighter",
      "type": "planting",
      "label": "Lily — Starfighter",
      "x": 0.50,
      "y": 0.74,
      "entity_ref": "life-brain:planting:planting-z7-lily-starfighter",
      "visible": true,
      "group": "plantings"
    }
  ]
}
```

---

### Field Reference

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `version` | integer | ✅ | Schema version. Increment on breaking changes. |
| `base_image` | string | ✅ | Filename of the base satellite image. |
| `image_size` | [int, int] | ✅ | `[width, height]` in pixels. Used for validation only — rendering uses CSS percentages. |
| `anchors[]` | array | ✅ | Registration marks for image re-fetch validation. Min 3. |
| `anchors[].id` | string | ✅ | Unique slug. |
| `anchors[].label` | string | ✅ | Human-readable description of the physical feature. |
| `anchors[].x` | float | ✅ | 0.0–1.0 from left edge. |
| `anchors[].y` | float | ✅ | 0.0–1.0 from top edge. |
| `pins[]` | array | ✅ | All entity pins on the map. |
| `pins[].id` | string | ✅ | Unique slug. Format: `{type}-{descriptor}`. |
| `pins[].type` | string | ✅ | One of: `irrigation_zone`, `planting`, `weed_treatment`, `pest_treatment`, `light_fixture`, `audio_zone`, `observation`. |
| `pins[].label` | string | ✅ | Display label shown in Tooltip and detail header. |
| `pins[].x` | float | ✅ | 0.0–1.0 from left edge. |
| `pins[].y` | float | ✅ | 0.0–1.0 from top edge. |
| `pins[].entity_ref` | string | ✅ | Reference to the entity data. Format: `{source}:{collection}:{id}`. Sources: `rachio`, `ha`, `life-brain`. |
| `pins[].visible` | boolean | ✅ | Whether this pin renders on the map. Used for layer filtering. |
| `pins[].group` | string | optional | Category group for layer toggle UI: `irrigation`, `plantings`, `treatments`, `lights`, `audio`, `observations`. |

**Coordinate rendering:**
```
pixel_x = Math.round(x * image_size[0])   // e.g., 0.52 * 512 = 266px
pixel_y = Math.round(y * image_size[1])   // e.g., 0.68 * 512 = 348px

css_left = `${(x * 100).toFixed(2)}%`     // "52.00%"
css_top  = `${(y * 100).toFixed(2)}%`     // "68.00%"
```

**Validation rules:**
- `x` and `y` must be in `[0.0, 1.0]` — reject values outside this range
- `id` must be unique across the entire `pins[]` array and `anchors[]` array
- `entity_ref` format must match `{source}:{collection}:{id}` — no free-form strings
- `type` must be one of the defined enum values

---

### `entity_ref` Format

| Source | Format | Example |
|--------|--------|---------|
| Rachio zone | `rachio:zone:{zone_id}` | `rachio:zone:7` |
| HA entity | `ha:{domain}.{entity_id}` | `ha:light.back_patio_string_lights` |
| life-brain planting | `life-brain:planting:{id}` | `life-brain:planting:planting-z7-lily-starfighter` |
| life-brain treatment | `life-brain:treatment:{id}` | `life-brain:treatment:weed-2026-05-01-tenacity-pre` |
| life-brain observation | `life-brain:observation:{date}` | `life-brain:observation:2026-05-01` |

---

## Section 4 — Named Areas

The named area taxonomy maps to Rachio zone names, known planting areas, and physical sections of the property. Named areas are referenced in entity schemas as the `area` field.

**Use:** When a pin has a named area ref, the detail drawer can show "all entities in this area." Named areas also serve as filter/grouping values in the layer toggle UI.

---

### Front Yard

| Area ID | Description | Rachio Zone | Notes |
|---------|-------------|-------------|-------|
| `front-yard-street` | Strip along the street curb, front of property | Zone 1 — Front Yard Street | Exposed to road, salt/drought stress |
| `front-yard-hill` | Sloped area between street and house | Zone 2 — Front Yard Hill | Hill terrain, runoff issues |
| `front-yard-garden` | Planting bed in front of the house | Zone 3 — Front Yard Garden | Most visible from street |

---

### Side Yards

| Area ID | Description | Rachio Zone | Notes |
|---------|-------------|-------------|-------|
| `lower-side-lawn` | Side yard lawn area, lower grade | Zone 4 — Lower Side Lawn | New sod + seed area; establishment in progress |
| `side-garden-upper` | Upper side garden (Zone 4 coverage) | Zone 4 | Goatsbeard, mystery trees (unidentified white spring blooms), hostas |
| `side-garden-lower` | Lower side garden (Zone 4 coverage) | Zone 4 | Willow-like shrub, Zinnias + annual seeds |

---

### Backyard

| Area ID | Description | Rachio Zone | Notes |
|---------|-------------|-------------|-------|
| `backyard-right` | Right (east) section of backyard | Zone 5 — Backyard Right | Open lawn |
| `backyard-left` | Left (west) section of backyard | Zone 6 — Backyard Left | Open lawn, clover patches noted May 2026 |
| `backyard-front` | Frontmost section of backyard (near house) | Zone 7 — Backyard Front | Sun Garden Build beds; peonies, daylilies, lilies; drip line supplement |

---

### Structures & Hardscape

| Area ID | Description | Notes |
|---------|-------------|-------|
| `retaining-wall-south` | South-facing retaining wall bed | Sun Garden Build — 50ft section; Lily mix, peonies, hollyhocks, daylilies. Full sun. |
| `retaining-wall-north` | North-facing retaining wall bed | Sun Garden Build — second 50ft section. Full sun. |
| `patio` | Main outdoor patio | Hardscape. Lights, audio zone. No lawn/planting entities but light + audio pins here. |

---

### Zone 8 (Disabled)

| Area ID | Description | Notes |
|---------|-------------|-------|
| `zone-8-disabled` | Zone 8 — currently disabled | Unknown coverage area. Disabled in Rachio. Excluded from active maps until re-enabled. |

---

### Area Hierarchy Summary

```
Property
├── Front Yard
│   ├── front-yard-street      (Z1)
│   ├── front-yard-hill        (Z2)
│   └── front-yard-garden      (Z3)
├── Side Yards
│   ├── lower-side-lawn        (Z4)
│   ├── side-garden-upper      (Z4)
│   └── side-garden-lower      (Z4)
├── Backyard
│   ├── backyard-right         (Z5)
│   ├── backyard-left          (Z6)
│   └── backyard-front         (Z7)
├── Structures
│   ├── retaining-wall-south
│   ├── retaining-wall-north
│   └── patio
└── Inactive
    └── zone-8-disabled
```

---

### Named Area — Planting Inventory (Current as of 2026-05-02)

**Zone 7 / retaining-wall-south / retaining-wall-north** (Sun Garden Build beds):
- Lily — Starfighter (Costco, planted 2026-04-19)
- Lily — Vestaro (Costco, planted 2026-04-19)
- Lily — Palazzo (Costco, planted 2026-04-19)
- Lily — Marlon (Costco, planted 2026-04-19)
- Peonies (Costco, planted 2026-04-19)
- Hollyhocks (Costco, planted 2026-04-19, need stakes)
- Daylilies (Costco, planted 2026-04-19)

**Zone 4 / side-garden-upper**:
- Goatsbeard (perennial)
- Mystery tree ×2 (white spring flowers, unidentified — flag for ID)
- Hostas

**Zone 4 / side-garden-lower**:
- Willow-like shrub (unidentified species — flag for ID)
- Zinnias + annual seed mix (2026 season)

**Containers / patio** (not in ground, not in Rachio zones):
- Roseform Begonias × [quantity TBD] — container planted 2026-05-02, shaded location

---

*Data model compiled: 2026-05-02. Based on: lawn-care-plan.md, pest-control.md, Sun Garden Build.md, Rachio zone list, HA integration, Frigate/UniFi coordinate system patterns.*
