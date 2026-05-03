---
type: handoff
project: "[[Lawn Hub]]"
created: 2026-05-03
updated: 2026-05-03
status: complete
tags: [lawn-hub, handoff]
---

# Lawn Hub v1 — Handoff

Live at **https://dash.dmelsh.com/lawn**. Branch: `lawn-hub`.

## Secrets / Env Vars

| Secret | Location | Used by |
|---|---|---|
| **Rachio API Key** | Hard-coded in `backend/routers/lawn.py` constant `RACHIO_API_KEY` | `/api/lawn/zones`, `/api/lawn/zones/{id}/run` |
| **Rachio Device ID** | Hard-coded in same file as `RACHIO_DEVICE_ID` | All Rachio endpoints |
| **HA JWT** | File: `~/.openclaw/secrets/ha.env` (single-line JWT, `chmod 600`) — same one used by `routers/ha.py` | `/api/lawn/rain`, `/api/lawn/lights`, `/api/lawn/lights/{id}/toggle` |
| **Dashboard auth** | `~/.openclaw/workspace/memory/.dashboard-credentials` (existing, unchanged) | All `/api/lawn/*` endpoints (cookie-protected) |

> **Action:** Move Rachio key + device ID to `~/.openclaw/secrets/rachio.env` as a v1.1 hardening pass. Hard-coding works for v1 but drifts from the HA pattern.

## HA Entity Dependencies

| Entity | Used for | Notes |
|---|---|---|
| `binary_sensor.rachio_7a45b8_new_rain` | Rain badge in toolbar | Reads `state` (on/off) + `last_changed` |
| `light.*` (filtered) | Outdoor light pins | Filter matches any of: `patio`, `yard`, `garden`, `outdoor`, `exterior`, `porch`, `deck`, `landscape`, `string`, `garage`, `driveway` in entity_id or friendly_name |
| `media_player.*` | **Not yet wired** — planned for `audio_zone` pin type in v2 | |

Currently 1 outdoor light is auto-discovered: `light.hue_discover_outdoor_wall_1_2` (Side Gate Light). Add more by re-naming any other outdoor lights in HA so the friendly name contains one of the filter fragments.

## Rachio Dependencies

| Endpoint | Method | Used for |
|---|---|---|
| `/1/public/device/{device_id}` | GET | Live zone state (called every 30s while page is open) |
| `/1/public/zone/start` | PUT body `{id, duration}` | Run-zone action button |

7 active zones discovered via the device endpoint (Z1–Z7). Z8 disabled, excluded.

## Obsidian Paths Read/Written

| Path | Direction | Purpose |
|---|---|---|
| `/mnt/c/Obsidian/life-brain/Lawn Hub/plantings/*.md` | read | `/api/lawn/plantings` parses frontmatter |
| `/mnt/c/Obsidian/life-brain/Lawn Hub/treatments/weed/*.md` | read | `/api/lawn/treatments` (last 90 days filter) |
| `/mnt/c/Obsidian/life-brain/Lawn Hub/treatments/pest/*.md` | read | same |
| `/mnt/c/Obsidian/life-brain/Lawn Hub/observations/YYYY-MM-DD.md` | read + write | List + append observation entries |

The directories are auto-created on first observation write. Reads return empty lists if directories don't yet exist.

## Cron Jobs

**No new cron jobs added in v1.** All polling is browser-side (30 s interval while the `/lawn` page is open). When the tab is closed, no background work runs.

> v2 candidate: a daily cron to mirror today's Rachio device events into a daily observation file, so the historical view doesn't depend on Rachio's API window.

## Files Changed

```
A backend/pins.json                          (new — 8+ pins, 3 anchors)
A backend/routers/lawn.py                    (new — 11 endpoints)
M backend/main.py                            (1 import + 1 include_router + extended static-file list)
A frontend/src/pages/LawnPage.jsx            (new — full page, drawer, forms)
A frontend/src/lib/lawnApi.js                (new — fetch wrapper)
M frontend/src/App.jsx                       (1 import + 1 Route)
M frontend/src/components/Nav.jsx            (1 NAV entry)
```

## Add to ACC README

Insert this block after the existing Phase 6 section:

```markdown
### Phase 7 — Lawn Hub (v1, 2026-05-03)

A satellite-image map of the property with clickable pins for irrigation zones,
plantings, lights, and observations. Lives at `/lawn`.

**Pins** are stored in `backend/pins.json` (normalized 0–1 coords on the
512×512 satellite image). Click a pin → bottom-sheet drawer with live state
and one-tap actions.

**Live data sources:**
- Rachio Public API (zones, run history, run command)
- Home Assistant REST (rain sensor, outdoor lights, light toggle)
- Obsidian markdown (`life-brain/Lawn Hub/`) for plantings, treatments,
  observations

**Add a pin:** long-press (or right-click) anywhere on the map.

**Log an observation:** "+ Observation" button on the bottom toolbar appends a
heading to today's `observations/YYYY-MM-DD.md` in the vault.
```

## Open Questions for Dan

1. **Pin placement is estimated.** All 11 v1 pins are guesses based on a generic suburban-property layout. First yard walkthrough with phone in hand → long-press to drop precise pins, then delete the placeholder pins from `backend/pins.json`.
2. **Notion project page parent** — Lawn Hub is not yet mirrored to a Notion parent page. Phase 5 deferred.
3. **Mystery trees + shrub in Z4 side gardens** — species unidentified. Flag for spring ID walk; once identified, create `plantings/*.md` files and add map pins.
4. **Hollyhock stakes** — Sun Garden Build still needs them; not blocking the hub but will affect plant placement once installed.
5. **Rachio zone IDs** — the `/zones` response uses Rachio's UUID-style IDs (e.g., `c7e84130-75b2-4db9-a880-5cc47506fa34`), which are passed back to `/zone/start` as the `id` field. The current pin `entity_ref` format `rachio:zone:1` (numeric) is resolved client-side by matching `zone_number`. Consider switching to `rachio:zone:<UUID>` in `pins.json` for direct use without a lookup. **Confirmed working as-is.**
6. **Audio zone entity IDs** — outdoor Sonos players are not yet confirmed. The hub knows about `audio_zone` as a pin type, but no audio pins exist in v1 and the drawer renders a placeholder. Once the entity_ids are confirmed, add pins like:
   ```json
   { "id": "audio-backyard", "type": "audio_zone", "label": "Backyard Speaker",
     "x": 0.65, "y": 0.60, "entity_ref": "ha:media_player.backyard_speaker",
     "visible": true, "group": "audio" }
   ```
7. **v2 backlog:**
   - Websocket subscription to HA state (replace 30s polling)
   - Notion mirror of pin inventory
   - Weather radar overlay (NWS or weatherkit)
   - Cron job to snapshot Rachio events daily
   - Drag-to-reposition pins (currently delete + re-add)
   - Pin clustering when zoomed out (not needed at z20 / 512px)
   - Audio zone wiring + Sonos volume sliders

## Acceptance Verified

- ✅ `/lawn` route loads (HTTP 200, 8+ pins rendered, 4 categories)
- ✅ `GET /api/lawn/pins` returns 11 pins, 3 anchors, version 1
- ✅ `GET /api/lawn/zones` returns 8 Rachio zones (Z1 currently in a watering run — confirms live data, not cached)
- ✅ `GET /api/lawn/rain` returns rain sensor (`off`, last changed 2026-04-18)
- ✅ `GET /api/lawn/lights` returns 1 outdoor light (`light.hue_discover_outdoor_wall_1_2`)
- ✅ `GET /api/lawn/observations` returns empty list (dir not yet created — by design)
- ✅ `npm run build` passes (only chunk-size warning, no errors)
- ✅ Auth middleware protects all `/api/lawn/*` (returns 401 without cookie)
- ✅ Satellite image served at `/warson-property-satellite.jpg` (200 OK, image/jpeg)
- ✅ Backend running under systemd as `acc-backend.service`

## Next Action for Dan

1. Open https://dash.dmelsh.com/lawn on phone
2. Walk the property; long-press to drop a real pin everywhere a placeholder is wrong
3. Delete placeholder pins from `backend/pins.json` (or add a delete-pin flow as a v1.1 follow-up)
4. Decide on Rachio key location (current: hard-coded → recommend: `~/.openclaw/secrets/rachio.env`)
