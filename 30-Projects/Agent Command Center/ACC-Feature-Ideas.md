---
type: project
status: active
created: 2026-04-24
updated: 2026-04-24
tags: [openclaw, dashboard, research, ideas]
---

# Agent Command Center — Top 10 Feature Ideas
*Research compiled: 2026-04-24*

---

## Feature #1: Bio-Cognitive State Overlay

**What it is:** A cross-source panel that maps Oura readiness/HRV scores against calendar density and AI token spend (as a proxy for "how much thinking Dan did"). Surfaces a daily cognitive load forecast and flags mismatches — like when a low-readiness day has back-to-back meetings or heavy project work scheduled.

**Why valuable for Dan:** He wears an Oura ring every night but probably rarely checks if he's scheduling his hardest work on his worst recovery days. This makes that invisible pattern visible. With young kids and sleep deprivation in the mix, readiness variance is real. Even a simple "today is a 62 — light meetings only" nudge could reshape how he plans.

**Data sources / APIs needed:**
- Oura Cloud API v2 (readiness, HRV balance, sleep score, deep sleep duration)
- Google Calendar API (event count, meeting duration, back-to-back gaps)
- OpenRouter / OpenClaw cost logs (tokens-in as cognitive effort proxy)
- Withings (weight, optional — correlate with high-stress weeks)

**Build effort:** M

**Pros:**
- Uses data Dan already collects — zero new tracking required
- Genuinely novel cross-source correlation nobody else is doing for him
- Could surface actionable patterns: "post-travel weeks = 3 low-readiness days, avoid scheduling client calls"

**Cons / risks:**
- HRV/readiness is lagging data (yesterday's sleep affects today's score) — framing matters
- Risk of over-optimization anxiety if Dan treats every score as a hard gate

**Mini project plan:**
1. Add Oura pull to daily cron if not already piping readiness score to a JSON store
2. Build a `/api/bio-state` endpoint that joins today's Oura data with calendar event count for today+tomorrow
3. Build React widget: readiness gauge (0-100) + calendar density bar + 3-day trend sparkline
4. Add "mismatch alert" logic: readiness < 65 AND calendar events > 4 → surface yellow warning banner
5. Log 30 days of data to enable correlation charts (readiness vs token spend over time)

---

## Feature #2: Home Entropy Index

**What it is:** A single composite score (0–100) representing how much "accumulated disorder" exists across Dan's life systems right now. Rolls up: aging pending approvals (by days-overdue), HA device anomalies, overdue cron jobs, stale ROADMAP items, and unfiled Obsidian inbox notes. Higher score = more stuff needing attention.

**Why valuable for Dan:** He has 27 cron jobs, a rental property, two vaults, a home with active renos, and a toddler. Entropy accumulates invisibly. A single number lets him see at a glance whether he's on top of things or falling behind — without opening 6 apps. It's the "dashboard of the dashboard."

**Data sources / APIs needed:**
- OpenClaw approval queue (pending count + age of each item)
- Home Assistant REST API (device unavailable count, automation failures)
- Cron health log (last-run timestamps, failure flags)
- Obsidian vault scan (files in inbox/ folders older than 7 days)
- Notion API (overdue tasks)

**Build effort:** M

**Pros:**
- Acts as a forcing function — if entropy > 70, something's clearly slipping
- Can be turned into a streak/gamification mechanic: "0 entropy days this month: 4"
- Zero-overhead situational awareness without reading anything

**Cons / risks:**
- Weighting the components is subjective and will need tuning
- Could be gamed by dismissing approvals rather than resolving them — needs "dismissed without action" tracking

**Mini project plan:**
1. Define entropy formula: weight each component (approvals: 30pts, cron failures: 20pts, HA anomalies: 20pts, stale ROADMAP: 15pts, Obsidian inbox: 15pts)
2. Build `/api/entropy` endpoint that collects each component score and returns composite
3. Add entropy gauge to Overview tab — large dial, color-coded (green/yellow/red)
4. Store daily entropy score to DB for trend chart (7-day rolling)
5. Add breakdown tooltip: hover the gauge to see "where the entropy is coming from"
6. Wire in a cron alert: if entropy > 80 for 2 consecutive days, send Slack DM

---

## Feature #3: Financial Anomaly Radar

**What it is:** A Monarch Money-powered panel that monitors spending patterns, flags statistical outliers vs. rolling 90-day baseline, and cross-references unusual spend against calendar events (travel, contractor work, kids appointments). Surfaces a monthly cash flow forecast and "burn rate vs. plan" delta.

**Why valuable for Dan:** Self-managed rental + active home renos = lumpy, irregular expenses. Contractor payments can slip through unnoticed. This makes the financial picture legible alongside everything else instead of requiring a separate Monarch app check.

**Data sources / APIs needed:**
- Monarch Money (transactions, account balances, budget categories)
- Google Calendar (travel events, contractor-scheduled appointments as spend context)
- Withings/Oura optional (correlate health spend with wellness outcomes — stretch)

**Build effort:** L

**Pros:**
- Catches "was this charge expected?" before it's a mystery 3 weeks later
- Calendar correlation makes anomalies explainable vs. alarming ("oh, that's the HVAC payment")
- Rental property expenses vs. personal expenses can be tracked separately

**Cons / risks:**
- Monarch API access may be limited (no official public API — may need CSV export + scraping or unofficial API)
- Financial data in a dashboard is a security surface — needs auth controls

**Mini project plan:**
1. Audit Monarch API access: check for unofficial API or export automation options
2. Build transaction ingestion pipeline (daily pull, store in local SQLite)
3. Compute rolling 90-day category baselines; flag transactions > 2σ above baseline
4. Build calendar join: match transaction date ±3 days to calendar events with location/person context
5. Build React panel: anomaly list with calendar context tooltip, 30-day spend sparkline, and month-end forecast
6. Add rental property filter toggle to isolate personal vs. property expenses

---

## Feature #4: Embedded Natural Language Query Terminal

**What it is:** A chat interface embedded directly in the ACC that can answer questions about any of Dan's connected data sources in natural language. "What did I spend on home repairs in Q1?" hits Monarch. "What's my sleep trend since my last work trip?" joins Oura with Google Calendar. Results render inline as charts or bullet lists.

**Why valuable for Dan:** The ACC already has data from 10+ sources. Right now, answering a cross-source question requires opening 3 apps. This collapses that into one box. It's not a chatbot for chatting — it's a structured query interface for *his* data.

**Data sources / APIs needed:**
- All existing connected sources (Monarch, Oura, Withings, Google, HA, Notion, Obsidian)
- OpenRouter/Claude as the query reasoning layer
- OpenClaw tool system (existing data-fetching tools as backend)

**Build effort:** L

**Pros:**
- Eliminates "I need to check 3 apps" workflows — massive time-saver
- Natural language removes the need to remember which tab has what
- Can be scoped to specific data domains for precision ("only query health data")

**Cons / risks:**
- Latency: multi-source queries may take 5-15 seconds — UX must handle gracefully
- Hallucination risk on numerical data — must always show source citations

**Mini project plan:**
1. Build a `/api/query` endpoint that accepts natural language + routes to appropriate data fetchers
2. Define a tool manifest for the query agent: `get_oura(date_range)`, `get_monarch_transactions(category, range)`, `get_calendar_events(range)`, etc.
3. Use Claude with tool-calling to parse intent and fetch from correct sources
4. Return structured response: answer text + source data snippet + optional chart spec (Recharts JSON)
5. Build React chat panel with message history, loading states, and inline chart rendering
6. Add query shortcuts: pre-built buttons for "Weekly Summary", "Sleep This Month", "Rental P&L"

---

## Feature #5: Predictive Day Brief Widget

**What it is:** A nightly-generated situation report — 5-8 bullet points synthesizing tomorrow's calendar, Oura readiness forecast, weather, pending approvals, email priority queue, and any triggered alerts — surfaced as a persistent "Today's Brief" card on the ACC Overview tab. Auto-regenerates at midnight.

**Why valuable for Dan:** The morning brief already exists as a cron job, but it goes to... somewhere. This makes it the first thing visible when he opens the dashboard — a single card that answers "what matters today?" without clicking anything.

**Data sources / APIs needed:**
- Google Calendar (tomorrow's events)
- Oura API (readiness score)
- Weather API (wttr.in)
- OpenClaw approval queue (count + top item)
- Gmail (unread priority messages)
- OpenRouter (Claude to synthesize the brief)

**Build effort:** S

**Pros:**
- Reuses existing cron infrastructure — just pipes output to a dashboard endpoint
- Forces clarity: the brief *must* be brief, so it's highly scannable
- Can include a "Dan's focus intent for today" freetext field he fills out manually

**Cons / risks:**
- If the nightly cron fails, the card shows stale data — needs a timestamp + "regenerate" button
- Brief quality depends on prompt quality — will need iteration

**Mini project plan:**
1. Modify existing morning-brief cron to write JSON output to `/api/brief` endpoint in addition to current delivery
2. Define JSON schema: `{date, readiness, weather_summary, top_meetings: [], pending_approvals: N, priority_email: string, agent_note: string}`
3. Build React "Today's Brief" card for Overview tab
4. Add manual "Regenerate" button that calls Claude with fresh data
5. Add "Dan's intent" freetext input that persists for the day

---

## Feature #6: Project Velocity + Stall Detector

**What it is:** Tracks ROADMAP items and active projects across Obsidian vaults, Notion tasks, and git commit history. Calculates per-project "velocity" (activity in last 7 days) and surfaces items that have gone stale (no activity in N days). Shows a burndown-style chart for active projects.

**Why valuable for Dan:** He has a ROADMAP, 2 Obsidian vaults, Notion, and actual git repos — but no single view of "which projects are moving and which are stuck." Stall detection is especially useful given his time constraints: projects go dormant without a forcing function.

**Data sources / APIs needed:**
- Obsidian vault file modification timestamps (`ls -lt` or inotify)
- Notion API (task updated_at timestamps)
- Git log for relevant repos (`git log --since="7 days ago"`)
- OpenClaw ROADMAP.md (parsed markdown checklist)

**Build effort:** M

**Pros:**
- Makes invisible project stalls visible before they become abandoned projects
- Velocity metric is honest — shows if Dan is actually working on what he says matters
- Can prioritize the Approvals inbox: items related to stalled projects get boosted

**Cons / risks:**
- "Activity" ≠ "progress" — file edits aren't always forward motion
- Obsidian vault parsing needs careful scoping to avoid reading private notes

**Mini project plan:**
1. Build project registry: list of active projects with their data source mappings (e.g., "ACC dashboard" → git repo + Obsidian folder + Notion page ID)
2. Build `/api/velocity` endpoint that pulls last-activity timestamps from each source per project
3. Calculate velocity score: weighted sum of commit count + note edits + Notion task completions in last 7 days
4. Flag stalled projects: velocity = 0 for > 14 days
5. Build React panel: project list with velocity bars, stall warning badges, and sparkline history
6. Add "nudge" mechanic: stalled projects appear in Approvals inbox with "Start a session on [project]?" prompt

---

## Feature #7: Context Fragmentation Score

**What it is:** A daily score (0-100) measuring how fragmented Dan's calendar is — calculated from meeting block count, gap duration between meetings, back-to-back sequences, and total "deep work window" availability. Rendered as a "Focus Potential" meter for today and the next 3 days.

**Why valuable for Dan:** Consultant work + home projects + two young kids = calendar full of interruptions. Knowing that Thursday has 2 hours of continuous deep work available vs. Monday's 12 fragmented 20-minute slots helps Dan schedule cognitively demanding work when his calendar actually allows it.

**Data sources / APIs needed:**
- Google Calendar API (event blocks, durations, gaps)
- Oura readiness (overlay: high focus potential + high readiness = ideal deep work day)

**Build effort:** S

**Pros:**
- Pure calendar math — no new integrations needed
- Actionable: "reschedule this meeting to Thursday" becomes data-backed advice
- Can be wired into the Predictive Day Brief automatically

**Cons / risks:**
- Doesn't account for async work interruptions (Slack, email) which aren't in calendar
- Score is descriptive, not prescriptive — doesn't auto-suggest rescheduling

**Mini project plan:**
1. Build `/api/focus-score` endpoint: fetch calendar events for today + 3 days, compute gap analysis
2. Score formula: longest_continuous_block × 0.4 + gaps_count_penalty × 0.3 + back_to_back_penalty × 0.3
3. Add Oura readiness overlay when available
4. Build React component: 4-day "Focus Forecast" strip with color-coded scores
5. Wire into Predictive Day Brief: "Today's focus potential: 72/100 — 2.5h deep work window at 9am"

---

## Feature #8: Rental Property Command Panel

**What it is:** A dedicated tab (or collapsible card) that consolidates everything related to Dan's rental property: monthly P&L from Monarch, maintenance log with status tracking, rent received vs. expected, upcoming tasks, and a simple notes area for tenant interactions. Single pane of glass for the landlord side of life.

**Why valuable for Dan:** Self-managed rental means wearing a property manager hat occasionally. Right now that data is scattered across Monarch (expenses), a mental log (maintenance), and probably a notes app. This surfaces it in the same dashboard as everything else, making it impossible to let "landlord mode" go dark for too long.

**Data sources / APIs needed:**
- Monarch Money (transactions filtered by rental property category/account)
- Notion (maintenance log if stored there) or simple local SQLite
- Google Calendar (scheduled maintenance, lease renewal dates)
- Manual input for tenant notes / rent status

**Build effort:** M

**Pros:**
- Consolidates scattered rental management into one view
- P&L tracking monthly makes tax prep easier (export-ready by year end)
- Calendar integration means lease renewals and maintenance dates don't get lost

**Cons / risks:**
- Rent payment tracking may need manual input if tenant doesn't pay via trackable method
- Maintenance log schema needs upfront thought to be useful long-term

**Mini project plan:**
1. Define data schema: `rent_payments`, `maintenance_items` (status: open/in-progress/closed), `expenses`
2. Build local SQLite tables (or Notion DB) for maintenance log
3. Build Monarch filter to isolate rental-tagged transactions
4. Build React tab: 3-column layout — P&L summary | Maintenance board | Upcoming calendar items
5. Add quick-add form for maintenance items and rent payment confirmation
6. Add monthly P&L export button (CSV) for tax records

---

## Feature #9: Agent Replay Timeline

**What it is:** A chronological, expandable audit trail of every AI agent action taken — cron runs, decisions made, emails sent, approvals processed, knowledge graph updates — with per-action cost attribution, source conversation snippet, and outcome status. Think "git log" but for your AI agent's life.

**Why valuable for Dan:** After a work trip or a busy week, it's impossible to know what the agent did, why, and whether it went well. Agent Replay makes the black box transparent. It also catches silent failures ("the rental cron ran but filed nothing"), which are worse than noisy failures.

**Data sources / APIs needed:**
- OpenClaw daily logs (memory/YYYY-MM-DD.md)
- OpenClaw cost tracker (cost-tracker.py output)
- Cron job execution logs
- Approval queue history (accepted/dismissed/deferred with timestamps)

**Build effort:** M

**Pros:**
- Trust-builder: Dan can verify the agent is acting as intended without reading raw logs
- Cost attribution per action surfaces expensive operations (which crons burn the most?)
- Provides a natural "catch-up" interface after travel — skim the last 5 days in 2 minutes

**Cons / risks:**
- Log parsing from markdown daily notes is fragile — needs structured logging to be reliable
- Risk of information overload: needs good filtering (by type, cost, outcome) to stay useful

**Mini project plan:**
1. Standardize cron and agent action logging to emit structured JSON events to a local `agent-events.db`
2. Each event: `{timestamp, type, description, cost_usd, model, outcome, source_file}`
3. Build `/api/timeline` endpoint with pagination and filter params (type, date range, min_cost)
4. Build React timeline component: grouped by day, expandable items, cost badges, outcome icons
5. Add "catch-up mode": button that loads last N days of events and asks Claude to summarize in 5 bullets
6. Wire existing cost-tracker.py to emit to the same event DB

---

## Feature #10: Family Density Calendar

**What it is:** A read-only calendar overlay that combines Dan's work calendar, Andrea's calendar, family events, and Dan's travel schedule to produce a composite "family bandwidth" view — color-coded by density. Highlights weeks where Dan is traveling AND family has high-activity, and flags upcoming "protected family time" windows.

**Why valuable for Dan:** Andrea's tension with projects isn't random — it's worst during weeks when Dan is also mentally or physically away. This feature makes the collision points visible *before* they happen, so Dan can plan projects into genuine low-density windows and proactively protect high-density family weeks from project creep.

**Data sources / APIs needed:**
- Google Calendar API (Dan's personal + work calendar)
- Andrea's Google Calendar (andreaelamont@gmail.com — read access)
- Family calendar (already connected)
- Work travel inference: calendar events with travel keywords / airline names

**Build effort:** S

**Pros:**
- Turns an invisible household tension source into a plannable variable
- No new tracking — everything is already in Google Calendar
- Could prevent the "I didn't realize how busy that week was" conversation entirely

**Cons / risks:**
- Andrea would need to keep her calendar current for this to be accurate
- Needs careful framing — this is for Dan's situational awareness, not to "optimize" family time like a resource

**Mini project plan:**
1. Pull all connected calendars for a 4-week rolling window via Google Calendar API
2. Compute daily "density score": event count × duration weight + travel day multiplier
3. Build `/api/family-density` endpoint returning daily scores for each person + composite
4. Build React 4-week calendar heatmap: each day color-coded by composite density
5. Add "protect this week" toggle that blocks it from project scheduling suggestions
6. Wire into Predictive Day Brief: flag if this week is high-density

---

## Summary Table

| # | Feature | Effort | Wow Factor | Dan-Specific Value |
|---|---------|--------|-----------|-------------------|
| 1 | Bio-Cognitive State Overlay | M | ⭐⭐⭐⭐⭐ | Sleep + calendar mismatch detection |
| 2 | Home Entropy Index | M | ⭐⭐⭐⭐⭐ | Single number for "how behind am I?" |
| 3 | Financial Anomaly Radar | L | ⭐⭐⭐⭐ | Monarch + calendar correlation |
| 4 | Natural Language Query Terminal | L | ⭐⭐⭐⭐⭐ | Query 10 data sources in one box |
| 5 | Predictive Day Brief Widget | S | ⭐⭐⭐⭐ | Reuses existing cron, surfaces it prominently |
| 6 | Project Velocity + Stall Detector | M | ⭐⭐⭐⭐ | Makes stalled projects impossible to ignore |
| 7 | Context Fragmentation Score | S | ⭐⭐⭐ | Pure calendar math, immediately actionable |
| 8 | Rental Property Command Panel | M | ⭐⭐⭐⭐ | Consolidates landlord mode into one place |
| 9 | Agent Replay Timeline | M | ⭐⭐⭐⭐⭐ | Transparency + trust for AI agent activity |
| 10 | Family Density Calendar | S | ⭐⭐⭐⭐⭐ | Makes the Andrea tension variable visible |

### Recommended Build Order (bang-for-buck)
1. **Family Density Calendar** — S effort, high household impact
2. **Predictive Day Brief Widget** — S effort, reuses existing infra
3. **Context Fragmentation Score** — S effort, pure calendar math
4. **Home Entropy Index** — M effort, flagship overview feature
5. **Agent Replay Timeline** — M effort, trust + transparency
6. **Bio-Cognitive State Overlay** — M effort, most novel feature
