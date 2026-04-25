---
type: research
project: Agent Command Center
created: 2026-04-23
updated: 2026-04-23
tags: [projects, agent-command-center, components]
---

# Agent Command Center — Component Catalog (Phase 2)

Every widget/module the Command Center could include, with what it integrates, tech tradeoffs, effort sizing, and release tier (v1 / v2 / later).

**Legend**
- **Effort:** S = <2h · M = half day · L = 1–2 days
- **Tier:** v1 = MVP · v2 = first follow-up · later = nice to have

---

## Category 1 — UI shell and layout

### 1.1 · App shell (nav, layout, routing)
- **What:** Top nav + route system that multiple dashboards plug into. Sticky header, side panel area optional.
- **Integrates:** Nothing yet — it's the frame.
- **Tech options:**
  - **React 19 + Vite + react-router** — already installed. Pro: zero setup, pro: same React as `CompDashboard.jsx`. Con: styling is inline; no design system.
  - **Next.js + shadcn/ui** — nicer defaults, dark mode, typography. Con: second framework in the same repo.
  - **Vanilla HTML + htmx** — tiniest possible stack. Con: no momentum against React already in place.
- **Recommendation:** Stick with Vite + React. Add **Tailwind + shadcn/ui** on top — it installs in 10 min and gives a real design language without a framework switch.
- **Effort:** S · **Tier:** v1

### 1.2 · Theme + density
- **What:** Dark mode default, compact density (Dan is a power user — no big empty cards).
- **Tech:** Tailwind `dark:` + a single density variable.
- **Effort:** S · **Tier:** v1

### 1.3 · Mobile layout
- **What:** Single-column on phone, dashboard grid on desktop.
- **Why:** Dan travels. A phone-friendly approvals/inbox view is the highest-value mobile feature.
- **Effort:** S (if Tailwind from day 1) · **Tier:** v1

### 1.4 · Card primitive (`<Card title source>`)
- **What:** Standardized card wrapper. Every panel is a card. Declares `source`, renders a small view, hides when empty (exception-based).
- **Why:** Lovelace discipline. Keeps the dashboard composable as the surface area grows.
- **Effort:** S · **Tier:** v1

---

## Category 2 — Project state views

### 2.1 · ROADMAP progress card
- **What:** Parse `ROADMAP.md`, render by section (Immediate / 30-day / 60-day / 90-day / PFA / SB / OC), with completion checkmarks + overdue flag. Progress bar per section.
- **Integrates:** `~/.openclaw/workspace/ROADMAP.md`.
- **Tech:** Server-side `marked` or `gray-matter` on WSL2 Python/Node; or client-side `react-markdown`.
- **Effort:** S · **Tier:** v1

### 2.2 · Active home projects board
- **What:** Read from life-brain `40-Areas/Home.md` + Notion Home Projects DB (`syncprojects` already exists as `scripts/sync-home-projects.py`).
- **Integrates:** Notion API + Obsidian file.
- **Effort:** M · **Tier:** v1

### 2.3 · Rental ledger card
- **What:** Rent collected MTD/YTD, unpaid, maintenance items. Uses `rental.md` + life-brain rental ledger.
- **Integrates:** Obsidian file parse.
- **Effort:** M · **Tier:** v2

### 2.4 · Travel horizon card
- **What:** Upcoming trip countdown with SOP phase marker (T-7 / T-3 / T-1 / return). Pulls from `MEMORY.md` travel section + calendar.
- **Integrates:** Memory file + Google Calendar (`gog`).
- **Effort:** S · **Tier:** v1

### 2.5 · Project chat drop-in (talk to project context)
- **What:** Each project card has a "talk about this" button that opens a chat panel pre-loaded with that project's context.
- **Integrates:** OpenClaw gateway session API (Mannys-style).
- **Effort:** L · **Tier:** later

---

## Category 3 — Task / todo surface

### 3.1 · TODO.md rendering
- **What:** Parse `~/.openclaw/workspace/TODO.md` and render as interactive checklist grouped by section (Urgent / Near-Term / Finances / Home / Rental). Clicking a checkbox rewrites the file.
- **Integrates:** File read/write on WSL2.
- **Why:** The morning brief already references TODO.md; the dashboard should let Dan close items without opening a terminal or Obsidian.
- **Effort:** S (read-only) · M (write-back with commit) · **Tier:** v1 read-only, v2 write-back

### 3.2 · Notion tasks merge
- **What:** If Dan's Notion "Home Projects" or tasks DB has action items, merge them into the TODO view with provenance (🗂 Notion).
- **Integrates:** Notion API.
- **Effort:** M · **Tier:** v2

### 3.3 · Quick-add
- **What:** Single text input at top of TODO card. Typing + Enter appends to TODO.md (or sends to CoS via Slack: `quick-add: ...`).
- **Integrates:** File write + optional Slack webhook.
- **Effort:** S · **Tier:** v2

### 3.4 · Overdue escalator
- **What:** Items in TODO older than their noted due date get a red flag. Matches the morning brief's "overdue" logic.
- **Integrates:** Just TODO.md parsing.
- **Effort:** S · **Tier:** v1

---

## Category 4 — Agent chat interaction panel

### 4.1 · Slack-mirror read-only
- **What:** Show the last N messages from Dan's Slack DM with CoS. Read-only. Lets the dashboard show "what did CoS last say?" without opening Slack.
- **Integrates:** Slack Web API (`conversations.history`).
- **Tech options:**
  - **Direct Slack API from backend** — requires Dan to authorize a token.
  - **Slack RSS (deprecated)** — not viable.
- **Effort:** M · **Tier:** v2

### 4.2 · Send-to-CoS input
- **What:** Text box on the dashboard that posts a message to the Slack DM. Reply comes back through the existing Slack flow.
- **Integrates:** Slack `chat.postMessage`.
- **Effort:** S (given 4.1's auth) · **Tier:** v2

### 4.3 · Gateway-direct chat (Mannys pattern)
- **What:** Chat panel that talks directly to the OpenClaw gateway via `/api/sessions/:key/message`, bypassing Slack entirely.
- **Why:** Faster feedback loop for development. Doesn't pollute Slack history with test messages.
- **Integrates:** OpenClaw gateway `:18789`.
- **Effort:** L · **Tier:** later (only if Dan wants a Slack-free path)

### 4.4 · Conversation history / sessions browser
- **What:** List past Slack conversations grouped by day/topic, searchable.
- **Integrates:** Slack API or OpenClaw session state.
- **Effort:** L · **Tier:** later

---

## Category 5 — Cron and automation health

### 5.1 · Cron roster + last-run card
- **What:** Show every cron (morning brief, triage, PFA pulse, EOD, Sunday reset, Oura, Withings, heartbeat) with: schedule, last run, last status (✅/❌/⏳), last runtime.
- **Integrates:**
  - Source: `~/.openclaw/workspace/memory/cost-log.json` (has `job`, `ts`, `duration_s`) — near-perfect as-is.
  - Supplement: cron log dir (OpenClaw logs at `/tmp/openclaw/*.log` or journald).
- **Effort:** S (cost-log only) · M (cost-log + log tail) · **Tier:** v1

### 5.2 · Failure alert strip
- **What:** Red banner across the top if any cron errored in the last 24h. Click to expand stack trace / last 20 log lines.
- **Integrates:** Cron log tail.
- **Effort:** M · **Tier:** v1

### 5.3 · Heartbeat status
- **What:** "Last heartbeat: 18m ago · checks run: Gmail, HA, TODO" — confirms heartbeat-state.json is updating.
- **Integrates:** `memory/heartbeat-state.json`.
- **Effort:** S · **Tier:** v1

### 5.4 · Run-now button
- **What:** Manual trigger for any cron from the dashboard (with confirmation).
- **Integrates:** systemd / cron job shell-out via WSL2.
- **Risk:** Running morning brief at 3 PM sends a weird Slack message. Add a confirmation + "dry run" option.
- **Effort:** M · **Tier:** v2

### 5.5 · Cron editor
- **What:** Dashboard-side cron edit UI.
- **Why not:** Dan edits cron in a text file. This UI adds complexity without much win. Skip.
- **Effort:** L · **Tier:** later

---

## Category 6 — Run history and logs

### 6.1 · Run timeline (Langfuse-style trace viewer)
- **What:** Timeline view per day. Each cron run = a bar. Clicking expands to show: input (e.g. "today's calendar"), output (the Slack message that was sent), model, tokens, cost, duration.
- **Integrates:** cost-log.json + script output capture (need to add: scripts should write their output to `memory/runs/YYYY-MM-DD/job-name-ts.json`).
- **Effort:** M (UI) + M (instrument scripts) = L total · **Tier:** v1 (lite version) / v2 (full)

### 6.2 · Morning brief archive
- **What:** Reader for the last N morning briefs. Searchable. Useful when Dan thinks "wait, did the brief mention X yesterday?"
- **Integrates:** Slack DM history OR a new "archive the brief to file" script.
- **Effort:** M · **Tier:** v1

### 6.3 · Triage archive
- **What:** Same pattern for inbox triage. "What did the 12:30 PM triage flag?"
- **Integrates:** Triage script output capture.
- **Effort:** S (if 6.1 done) · **Tier:** v1

### 6.4 · Agent run detail page
- **What:** Deep-link to a single run. URL: `/run/:ts`. Shows full input/output/model/cost/prompts. Helpful when something broke.
- **Integrates:** Run archive.
- **Effort:** M · **Tier:** v2

### 6.5 · Session replay (full conversation inspection)
- **What:** Mannys-style — see every message in a gateway session.
- **Effort:** L · **Tier:** later (depends on gateway session storage)

---

## Category 7 — Cost and token tracking

### 7.1 · Daily spend card
- **What:** Today's cost, yesterday's, 7-day rolling. Split by model + by job. Matches the EOD wrap cost summary already being written.
- **Integrates:** `memory/cost-log.json`.
- **Effort:** S · **Tier:** v1

### 7.2 · Monthly spend trend
- **What:** Bar chart by day, area chart by model. Recharts already installed.
- **Integrates:** cost-log.json.
- **Effort:** S · **Tier:** v1

### 7.3 · Claude Max utilization
- **What:** "You ran X sessions on Max this month, equivalent to $Y at API pricing. Max subscription = $Z. Net savings: $(Z-Y)." Justifies the subscription at a glance.
- **Integrates:** cost-log.json (already tracks `is_max_plan`).
- **Effort:** S · **Tier:** v1

### 7.4 · OpenRouter reconciliation
- **What:** Pull OpenRouter's `/api/v1/activity` and diff vs. local cost-log. Flag undercounts.
- **Integrates:** OpenRouter API.
- **Effort:** M · **Tier:** v2

### 7.5 · Cost budget + burn rate alert
- **What:** Set a monthly budget. Show burn rate, project end-of-month spend.
- **Effort:** S · **Tier:** v2

### 7.6 · Per-cron unit cost
- **What:** "Morning brief costs $0.08 avg per run × 22 weekdays = $1.76/mo." Useful when evaluating whether to downgrade a cron to Flash.
- **Integrates:** cost-log.json.
- **Effort:** S · **Tier:** v1

---

## Category 8 — Data source connectors

### 8.1 · Notion connector
- **What:** Library to read Home Projects, Bills, Rental, Blue, Travel, Meals DBs.
- **Integrates:** Notion API with `ntn_S79073248198oOfq2U3MITSxd1TWK2oOT9Hl1OSKC1bcAd`.
- **Tech:** Use `@notionhq/client` in Node, or continue with Python `requests` (existing `scripts/sync-home-projects.py` shows the pattern).
- **Effort:** S (wrapper library) · **Tier:** v1

### 8.2 · Obsidian vault reader
- **What:** File walker for work-brain + life-brain. Reads frontmatter, tag index, backlinks for a given note.
- **Integrates:** `/mnt/c/Obsidian/{work,life}-brain/`.
- **Effort:** S · **Tier:** v1

### 8.3 · Slack connector
- **What:** Read DM history + post messages.
- **Integrates:** Slack Web API + bot token (needs setup if not already wired).
- **Effort:** M · **Tier:** v2

### 8.4 · Gmail / Calendar connector
- **What:** Shell-out to `gog` OR rewire via google-api-python-client.
- **Integrates:** `gog` CLI (already configured).
- **Effort:** S (subprocess) · **Tier:** v2

### 8.5 · OpenRouter usage connector
- **What:** Wrapper around `/api/v1/activity` + `/api/v1/generation/:id`.
- **Effort:** S · **Tier:** v2

### 8.6 · Home Assistant connector
- **What:** Wrap `http://100.94.72.30:8123/api/states` with a typed interface for Dan's entities (doors, lights, leak sensors, person.andrea, HVAC).
- **Integrates:** HA REST API + token in `~/.openclaw/secrets/ha.env`.
- **Effort:** S · **Tier:** v2

### 8.7 · Monarch connector
- **What:** Read `monarch_transactions_full.json` (already pulled by `scripts/monarch-fetch.py`).
- **Effort:** S · **Tier:** v2

### 8.8 · Health connector (Oura + Withings + Garmin)
- **What:** Read outputs of `scripts/oura-pull.py`, `scripts/withings-pull.py`, `scripts/garmin-pull.py`.
- **Effort:** S · **Tier:** v2

---

## Category 9 — Notifications surface

### 9.1 · Approvals inbox (the highest-ROI widget)
- **What:** Central list of anything CoS needs Dan to approve: travel SOP actions, financial thresholds crossed, auto-merge PRs, triage "action required" items, heartbeat escalations. Each has Accept / Decline / Snooze buttons.
- **Integrates:** Whatever source raised the approval. Requires a schema: every approval writes `memory/approvals/YYYYMMDD-HHMMSS-slug.json` with `{id, source, summary, actions, expires_at}`.
- **Why v1:** This alone can reduce Slack scroll burden by 50%+.
- **Effort:** M (UI) + M (retrofit 3–4 crons to write approvals) = L · **Tier:** v1

### 9.2 · Today's alerts feed
- **What:** Stream of all alerts fired in the last 24h: heartbeat pings, triage flags, cron errors, financial watchdog catches, HA anomalies.
- **Integrates:** Alert log.
- **Effort:** M · **Tier:** v1

### 9.3 · Snooze / dismiss history
- **What:** Track what Dan has dismissed so the same item doesn't re-surface. Matches "never show the same item twice."
- **Integrates:** Alert log + a tiny SQLite table.
- **Effort:** M · **Tier:** v2

### 9.4 · Push to phone
- **What:** If something critical fires while the dashboard is closed, push via Slack (already working) or via ntfy.sh / Pushover.
- **Effort:** S (Slack already done) · **Tier:** v2 (only if Slack-push ever feels insufficient)

---

## Category 10 — Auth and access control

### 10.1 · Localhost-only (MVP)
- **What:** Bind to `127.0.0.1:3737`. Dan accesses from the WSL2 host only.
- **Risk:** None. He's the only user on DESKTOP-AU0QVMV.
- **Effort:** S · **Tier:** v1

### 10.2 · Tailscale exposed
- **What:** Bind to `0.0.0.0:3737`, rely on Tailscale ACLs to restrict access to Dan's devices only. He already has Tailscale wired across laptop + phone + seedbox + NAS + HA.
- **Risk:** If Tailscale is compromised, the dashboard is too. Mitigate with 10.3.
- **Effort:** S · **Tier:** v1 (recommended from day 1 so mobile Just Works)

### 10.3 · Username/password (Mannys pattern)
- **What:** scrypt-hashed local password file. Session cookie (HttpOnly, 24h TTL) or bearer token. First-load creates the credentials.
- **Integrates:** Pure local — no third-party auth provider.
- **Effort:** M · **Tier:** v2

### 10.4 · SSO / OAuth
- **What:** Sign-in with Google.
- **Why not:** Dan is one user. Massive overkill.
- **Effort:** M · **Tier:** later (likely never)

---

## Summary grid

| # | Component | v1 | v2 | later | Effort |
|---|---|---|---|---|---|
| 1.1 | App shell | ✅ | | | S |
| 1.2 | Theme | ✅ | | | S |
| 1.3 | Mobile | ✅ | | | S |
| 1.4 | Card primitive | ✅ | | | S |
| 2.1 | ROADMAP card | ✅ | | | S |
| 2.2 | Home projects board | ✅ | | | M |
| 2.3 | Rental ledger | | ✅ | | M |
| 2.4 | Travel horizon | ✅ | | | S |
| 2.5 | Project chat | | | ✅ | L |
| 3.1 | TODO.md render (r/o) | ✅ | ✅ (w) | | S/M |
| 3.2 | Notion tasks merge | | ✅ | | M |
| 3.3 | Quick-add | | ✅ | | S |
| 3.4 | Overdue escalator | ✅ | | | S |
| 4.1 | Slack mirror | | ✅ | | M |
| 4.2 | Send to CoS | | ✅ | | S |
| 4.3 | Gateway chat | | | ✅ | L |
| 4.4 | Session history | | | ✅ | L |
| 5.1 | Cron last-run | ✅ | | | S/M |
| 5.2 | Failure alert | ✅ | | | M |
| 5.3 | Heartbeat status | ✅ | | | S |
| 5.4 | Run-now button | | ✅ | | M |
| 6.1 | Run timeline | ✅ (lite) | ✅ | | M/L |
| 6.2 | Brief archive | ✅ | | | M |
| 6.3 | Triage archive | ✅ | | | S |
| 6.4 | Run detail page | | ✅ | | M |
| 6.5 | Session replay | | | ✅ | L |
| 7.1 | Daily spend | ✅ | | | S |
| 7.2 | Monthly trend | ✅ | | | S |
| 7.3 | Max utilization | ✅ | | | S |
| 7.4 | OpenRouter reconcile | | ✅ | | M |
| 7.5 | Budget alert | | ✅ | | S |
| 7.6 | Per-cron cost | ✅ | | | S |
| 8.1 | Notion connector | ✅ | | | S |
| 8.2 | Obsidian reader | ✅ | | | S |
| 8.3 | Slack connector | | ✅ | | M |
| 8.4 | Gmail/Cal connector | | ✅ | | S |
| 8.5 | OpenRouter connector | | ✅ | | S |
| 8.6 | HA connector | | ✅ | | S |
| 8.7 | Monarch connector | | ✅ | | S |
| 8.8 | Health connector | | ✅ | | S |
| 9.1 | **Approvals inbox** | ✅ | | | L |
| 9.2 | Alerts feed | ✅ | | | M |
| 9.3 | Snooze history | | ✅ | | M |
| 9.4 | Push notifications | | ✅ | | S |
| 10.1 | Localhost-only | ✅ | | | S |
| 10.2 | Tailscale expose | ✅ | | | S |
| 10.3 | Password auth | | ✅ | | M |
| 10.4 | OAuth | | | ✅ | M |

**v1 count:** 22 components · estimated total effort ≈ 7–10 build days if done in one sweep, 2–3 weekends if paced.
**v2 count:** 14 components.
