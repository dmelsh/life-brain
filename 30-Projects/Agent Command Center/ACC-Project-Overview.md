---
type: project
status: active
created: 2026-04-24
updated: 2026-04-24
outcome: Personal operations dashboard live at dash.dmelsh.com
target_date: 2026-05-01
area: "[[Home]]"
tags: [openclaw, dashboard, home-cos, infrastructure]
---

# Agent Command Center (ACC)

> Personal operations dashboard for the OpenClaw Home CoS system. Built 2026-04-24.

**Live:** https://dash.dmelsh.com  
**Password:** in `~/.openclaw/secrets/dashboard.env`  
**Source:** `~/projects/acc/`  
**Start:** `bash ~/projects/acc/start.sh`

---

## What It Does

Replaces "scroll Slack to find what needs attention" with a single browser view, accessible from any device (phone, laptop, tablet) via public HTTPS.

---

## Pages

| Page | Route | What You See |
|------|-------|-------------|
| Overview | `/` | HA house status, Travel countdown, TODO, ROADMAP progress |
| 🏠 Home | `/home` | Animated task cards from life-brain (Lawn, Maintenance, Projects, Pest, Rental, Finances, Family, Trips) |
| Cost | `/cost` | Daily spend, MAX plan utilization, 30-day trend, per-cron breakdown |
| 💼 Comp | `/comp` | Compensation: salary, RSU, commission, tax, 401k charts |
| Crons | `/crons` | All 27 crons with status, last run, next run, failure strip, ▶ Run button |
| Approvals | `/approvals` | Pending action items with Accept/Dismiss/Defer buttons |
| History | `/history` | 30-day activity heatmap, daily notes archive, run log |
| Family | `/family` | Family context |
| Timeline | `/timeline` | Agent event log with cost breakdown |
| 🌌 Universe | `/graph` | WebGL 3D knowledge graph with live job ships |

---

## Architecture

**Backend:** FastAPI (Python), port 3737, serves built React app  
**Frontend:** Vite + React 19 + Tailwind + shadcn/ui  
**Auth:** scrypt password → JWT session cookie (24h)  
**Hosting:** Cloudflare Tunnel `dd370490` → WSL2 backend via portproxy  

```
Internet → Cloudflare (dash.dmelsh.com) → NAS tunnel → Windows Tailscale 100.76.105.68:3737
→ Windows portproxy → WSL2 172.18.158.95:3737 → FastAPI backend
```

---

## Key Files

| Path | Purpose |
|------|---------|
| `~/projects/acc/start.sh` | One-command launcher |
| `~/projects/acc/backend/main.py` | FastAPI entry point |
| `~/projects/acc/backend/auth.py` | Password auth + JWT middleware |
| `~/projects/acc/README.md` | Full technical docs |
| `~/.openclaw/workspace/scripts/create-approval.py` | Create approval from any cron |
| `~/.openclaw/workspace/scripts/save-run-output.py` | Archive cron output |
| `~/.openclaw/workspace/memory/approvals/` | Pending approval files |

---

## Data Sources Wired

- `cost-log.json` — token spend per job
- `heartbeat-state.json` — last heartbeat checks
- `TODO.md` + `ROADMAP.md` — project/task state
- `MEMORY.md` — travel info
- `openclaw cron list --json` — live cron roster
- Home Assistant REST API — door/leak/HVAC/presence
- `memory/YYYY-MM-DD.md` — daily notes archive (17 notes from Mar 18)

---

## Cloudflare Setup

- **Tunnel ID:** `dd370490-6fc6-4baa-b276-cea834c200a7`
- **Hostname:** `dash.dmelsh.com → http://100.76.105.68:3737`
- **NAS Docker:** `/volume1/docker/cloudflared/compose.yaml` (network_mode: host)
- **WSL2 connector:** `~/.local/bin/cloudflared` (started by start.sh)
- **API token:** `~/.openclaw/secrets/cloudflare.env` (Zero Trust Write, DNS Write, 1yr)

⚠️ **WSL2 portproxy caveat:** WSL2 internal IP changes on restart. After WSL2 restart, run in admin PowerShell:
```powershell
netsh interface portproxy delete v4tov4 listenport=3737 listenaddress=100.76.105.68
netsh interface portproxy add v4tov4 listenport=3737 listenaddress=100.76.105.68 connectport=3737 connectaddress=<new-wsl2-ip>
```
Or just run `bash ~/projects/acc/start.sh` which should handle this going forward.

---

## Build Log — 2026-04-24

All 6 phases + public hosting completed in a single day.

| Time | Milestone |
|------|-----------|
| 08:50 AM | Phase 1 kicked off (Claude Code, session nimble-basil) |
| 09:00 AM | Phase 1 ✅ — FastAPI + Cost tab live |
| 12:50 PM | Phase 2 kicked off |
| 12:55 PM | Phase 2 ✅ — Cron health + heartbeat |
| 14:38 PM | Phase 3 kicked off |
| 14:44 PM | Phase 3 ✅ — Approvals inbox |
| 15:03 PM | Phase 4 kicked off |
| 15:08 PM | Phase 4 ✅ — Overview (ROADMAP/TODO/Travel) |
| 15:25 PM | Phase 5 kicked off |
| 15:30 PM | Phase 5 ✅ — History + daily notes archive |
| 16:39 PM | Phase 6 kicked off |
| 16:49 PM | Phase 6 ✅ — Auth + HA status + run-now + TODO write-back |
| 20:41 PM | dash.dmelsh.com live ✅ |

---

## v2 Backlog

- [ ] Fix WSL2 portproxy auto-update on restart
- [ ] systemd service for backend (auto-start)
- [ ] Retrofit crons to write approvals (morning brief, triage, EOD)
- [ ] Retrofit crons to save run outputs (history archive)
- [ ] TODO write-back checkbox (mark done from dashboard)
- [ ] Smart TODO update → CoS review → cascade to Notion/life-brain
- [ ] Notion tasks merge into TODO card
- [ ] Slack mirror (last N CoS messages in dashboard)
- [ ] OpenRouter cost reconciliation
- [ ] Add more crons to cost-log.json

---

## Related Files in This Vault

- [[01-research]] — Market research (YouTube, GitHub, Langfuse, Lovelace)
- [[02-components]] — Full component catalog (45 widgets, v1/v2/later)
- [[03-recommendation]] — Architecture decision (Option A/B/C analysis)

---

## Dev Log

### 2026-04-25 — Home Dashboard Sprint

**Added:**
- `/home` tab with 9 animated card groups pulling from life-brain markdown
- Responsive nav (hamburger/sidebar) replacing broken horizontal tab bar
- Task actions: complete (writes back to .md), defer, edit, move between cards, add
- Custom section cards (add/delete, persist in home-order.json)
- `/comp` tab merged in from separate dashboard/ app
- 4 new data sources: Rental, Finances, Family, California Trip
- Created `30-Projects/Lawn Care 2026.md` from Home Pulse tasks
- Fixed Timeline tab crash (date_range object → formatted string)
- Claude Code overnight: design unification + health/weather widgets on Overview

**Architecture notes:**
- Home data API: `GET /api/home/files` → parses 8 markdown files
- Write-back: `/api/home/toggle`, `/api/home/edit`, `/api/home/remove`, `/api/home/append`
- UI order/custom sections: `/api/home/order` ↔ `home-order.json`
- All markdown files remain source of truth — dashboard is display + interaction layer only
