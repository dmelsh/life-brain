---
type: project
status: done
created: 2026-04-24
updated: 2026-04-24
outcome: All 6 phases built + dash.dmelsh.com live
area: "[[Home]]"
tags: [openclaw, dashboard, build-log]
---

# ACC Build Log — 2026-04-24

Full session log of the Agent Command Center build day.

---

## Context

Following the overnight research session (Apr 23), Dan approved *Option B (Balanced)* architecture at 8:49 AM. Build started immediately using Claude Code for each phase, OpenClaw (CoS) as the interface and coordinator.

---

## Phase 1 — FastAPI Skeleton + Cost Tab
**Session:** nimble-basil | **Start:** 08:50 | **Done:** 09:00

**Built:**
- `backend/main.py` — FastAPI app, CORS, static file serving
- `backend/routers/cost.py` — `/api/cost/entries`, `/api/cost/summary`
- `backend/routers/overview.py` — `/api/overview`
- `frontend/` — Vite + React 19 scaffold with Tailwind + shadcn/ui
- `CostPage.jsx` — DailySpendCard, MaxUtilizationCard, MonthlyTrendCard (recharts area chart), PerCronCostCard
- `DashCard.jsx` — card primitive with source label + hidden prop

**Notable:** cost.py handles both Unix epoch AND ISO string timestamps in cost-log.json — future-proof across log format changes.

---

## Phase 2 — Cron Health + Heartbeat Status
**Session:** tide-mist | **Start:** 12:50 | **Done:** 12:55

**Built:**
- `backend/routers/crons.py` — `/api/crons` (shells `openclaw cron list --json`), `/api/heartbeat`, `/api/alerts` (stub)
- `CronsPage.jsx` — FailureAlertStrip (errors-only), CronRosterCard (27 jobs, badges, relative times), HeartbeatStatusCard

**Discovery:** `openclaw cron list --json` returns rich structured data including consecutive_errors, last run status, duration_ms, next run time. Used directly.

**Crons currently erroring:** Daily Oura Pull, Daily Withings Pull, Work Brain Daily Note, PFA Weekly Pulse.

---

## Phase 3 — Approvals Inbox
**Session:** rapid-wharf | **Start:** 14:38 | **Done:** 14:44

**Built:**
- Approvals JSON schema + `memory/approvals/` directory
- `scripts/create-approval.py` — CLI helper for crons to surface approvals
- `backend/routers/approvals.py` — full CRUD + auto-expiry
- `ApprovalsPage.jsx` — cards with action buttons, pending count nav badge, empty state
- 3 seed approvals (PFA Weekly Pulse failure, Oura ring no data, Monarch cleanup)

**Decision:** Approvals are the highest-value feature — cuts Slack scroll burden by surfacing actionable items in one place.

---

## Phase 4 — Overview Page (ROADMAP + TODO + Travel)
**Session:** glow-daisy | **Start:** 15:03 | **Done:** 15:08

**Built:**
- `backend/routers/content.py` — `/api/roadmap` (parses ROADMAP.md), `/api/todo` (parses TODO.md, flags overdue), `/api/travel` (reads MEMORY.md + gog calendar)
- `OverviewPage.jsx` — TravelHorizonCard (25 days to California, SOP phase badge), TODOCard (grouped by section, urgent expanded), ROADMAPCard (overall progress bar, section breakdown)

**Current state:** 28/35 ROADMAP items complete.

---

## Phase 5 — History + Daily Notes Archive
**Session:** plaid-seaslug | **Start:** 15:25 | **Done:** 15:30 (SIGTERM after completion)

**Built:**
- `backend/routers/history.py` — daily notes, runs, timeline endpoints
- `HistoryPage.jsx` — 30-day activity heatmap, DailyNotesList (react-markdown), RunsList
- `scripts/save-run-output.py` — helper for archiving cron outputs
- `memory/runs/` — directory for future run archives

**Note:** 17 daily memory notes (Mar 18 → Apr 23) available immediately in the archive.

---

## Phase 6 — Auth + HA + Run-Now + TODO Write-back
**Session:** nova-mist | **Start:** 16:39 | **Done:** 16:49 (SIGTERM after completion)

**Built:**
- `backend/auth.py` — scrypt password, JWT session cookie, auth middleware
- `backend/routers/ha.py` — Home Assistant status (doors, leak, people, temps)
- Login page, nav sign-out, auth status check on app load
- TODO write-back: checkbox toggle (PATCH), smart update with CoS review (POST → approval)
- `scripts/apply-todo-update.py` — apply reviewed TODO change + cascade
- Run-now button on each cron row (confirm dialog → `openclaw cron trigger <id>`)

**Dashboard password generated:** `REwwcUfGslVG` (saved to secrets)

---

## Public Hosting — dash.dmelsh.com
**Time:** 18:04–20:41

**Problem:** WSL2 Tailscale IP not reachable from NAS cloudflared container.

**Solution path:**
1. Tried: NAS cloudflared → WSL2 Tailscale IP (100.67.96.53) → ❌ double-NAT
2. Tried: NAS cloudflared → Windows Tailscale IP (100.76.105.68) → ❌ NAS still couldn't reach it
3. Added Windows portproxy (admin PowerShell): `100.76.105.68:3737 → 172.18.158.95:3737`
4. Added Windows Firewall rule: inbound TCP 3737
5. Still failed: NAS can't reach Windows Tailscale IP
6. **Fix:** Installed cloudflared directly in WSL2 (`~/.local/bin/cloudflared`) with same tunnel token → WSL2 connector registered at STL → routes directly → ✅

**Final state:** Both NAS connector + WSL2 connector active. WSL2 connector reliably serves `dash.dmelsh.com → localhost:3737` (via 100.76.105.68:3737 portproxy).

**Cloudflare API token:** Created but wrong permissions (Zero Trust Write ≠ Tunnel hostname write). Manual hostname config required via dashboard.

---

## Open Items After Today

1. **WSL2 portproxy breaks on restart** — need Task Scheduler job or startup script to update portproxy rule when WSL2 IP changes
2. **Cron retrofit** — morning brief, triage, EOD should write approvals + save run outputs
3. **TODO write-back** — Phase 6 built the backend, need to wire frontend checkbox
4. **systemd service** — auto-start backend on WSL2 boot

---

## Decisions Made Today

| Decision | Rationale |
|----------|-----------|
| Option B (Balanced) architecture | A = too limited, C = over-built |
| FastAPI backend | Matches existing Python scripts |
| Card-first UI (Lovelace model) | Exception-based surfacing |
| Phase priority: cost → crons → approvals → content → history → auth | Most-valuable-first |
| Phase 6 priority: auth → TODO write-back → run-now → HA | Dan's explicit order |
| Smart TODO update → CoS review flow | Edit triggers approval → I cascade changes |
| Public hosting at dash.dmelsh.com | Dan wants access from phone without Tailscale |
| Option A hosting (NAS-based tunnel) | For future CoS access to dashboard |
| WSL2 cloudflared connector | Bypasses NAS→WSL2 Tailscale routing problem |
