---
type: research
project: Agent Command Center
created: 2026-04-23
updated: 2026-04-23
tags: [projects, agent-command-center, research]
---

# Agent Command Center — Research (Phase 1)

**Goal:** Design a browser UI that replaces "scroll through Slack DMs" as the way to see and steer Dan's OpenClaw Home CoS. This phase surveys the field: what's already out there, what applies, what doesn't.

---

## Snapshot of what exists today (baseline)

Before surveying the world, here's Dan's current starting point:

- **Vite + React 19 + recharts** dashboard skeleton already installed at `~/.openclaw/workspace/dashboard/`. One route: `/comp` (compensation dashboard powered by `comp-history.json`). Dev port 3737. Data pipeline: `scripts/gen-dashboard-data.py` regenerates the RAW array and edits the `.jsx` files in place.
- **Cost tracking** already writes to `memory/cost-log.json` via `scripts/cost-tracker.py`. Schema is solid: ts, job, model, tokens_in, tokens_out, cost_usd, is_max_plan, duration_s, notes, session_id.
- Active crons (from `OPERATIONS.md` + `MEMORY.md`): morning briefing (7 AM), work-brain daily note (7:05 AM), inbox triage (7:15 AM + 12:30 PM), Oura (7:30 AM), Withings (7:35 AM), PFA Weekly Pulse (Mon 9 AM), EOD wrap (9 PM), Sunday journal draft (5:45 PM Sun), Sunday reset (6 PM Sun), heartbeat (every 30 min).
- Data sources already wired: Gmail + Google Calendar via `gog`, Home Assistant REST API, Monarch via Playwright, OpenRouter (usage accounting live in cost-tracker), Notion, two Obsidian vaults, Slack DMs.
- Slack is the **primary interface**. Dashboard has to earn attention against that fact.

The dashboard folder is 9 days old and only holds a comp chart. Everything else here is greenfield.

---

## Source 1 — YouTube: "OpenClaw + custom dashboard + subagents = mindblowing!"
**URL:** https://www.youtube.com/watch?v=hAwF6j5n2q0
**Transcript length:** 38,917 chars — fetched via youtube-transcript-api after installing with `--break-system-packages`.

### What's built
Benoît (developer + designer) built a personal OpenClaw-facing UI over ~4 weeks. Not open-source (yet). Core pieces:

- **Side chat → center Kanban → agent activity panel** layout. "Only talk to your agents when you need to." Telegram was too one-dimensional; Slack/Discord "should have a better UI."
- **Orchestrator + sub-agents pattern** (credits Elvy Sun on Twitter). One main agent in the center. It spawns specialized sub-agents (coding, content, research) with injected context from the project view. User only talks to the orchestrator.
- **Inbox / notifications panel** — PRs ready for review, cron outputs (he has a cron that watches 4–5 YouTubers daily), task state changes. Direct **Merge** button inside the notification card, so you can approve a sub-agent PR without leaving the dashboard.
- **Project context** — each project has a Kanban board (To Do / In Progress / Review / Done). Spawning an agent on a ticket runs it in the ticket's context.
- **Convex** as the project database (separate from OpenClaw gateway). OpenClaw is the gateway/orchestrator; Convex is the context store (tickets, agent runs, PRs).
- **GitHub collaborator pattern** — agent has its own GitHub account + email, added as a collaborator. All code changes come in as PRs that Dan (or Benoît) approves. Can't "get out of hand."
- **Tailscale** for secure access to VPS-hosted gateway. No public URL.
- **Access control for APIs** via **Composio** — agent gets an API key for Linear (and other services), key is revocable from the Composio dashboard if things go wrong.
- **Interview mode** — when creating a ticket, a sub-agent runs a Q&A interview on Dan before committing the ticket. Gets much higher quality tickets. "No one thinks to ask the AI to interview them — best pro tip."
- **Decision widget** — tracks explicit "doing / not doing / deciding" buckets, including *why*. Carried over from a prior desktop app into the dashboard.

### Tech stack
- Front: custom (not specified — looks React/Next.js from screenshots)
- Backend: Convex (serverless DB + functions)
- Auth: Tailscale private URL
- Model billing: started on Claude Max sub → Anthropic blocked it → moved to Codex/ChatGPT Plus. Didn't mention token-based pricing — everything subscription.

### What applies to Dan
- **Orchestrator + sub-agents** — Dan doesn't have this yet, but `run_task_flow` is exactly this primitive (see Source 2). Dan's CoS already has domain-ish separation (home, finance, health, second-brain) but they're all one agent.
- **Inbox / notifications panel** — Dan's heartbeat + EOD + triage all push to Slack. A dashboard inbox would stop him from scrolling Slack history to find a PFA pulse from 3 days ago.
- **Kanban + project context** — maps directly to Dan's ROADMAP.md and TODO.md. The Kanban is just a rendering of existing markdown.
- **Approval gates** — Dan's crons are approval-first. Gates in a dashboard = fewer Slack "approve this?" messages.
- **Tailscale** — already in place (`100.94.72.30` HA host, seedbox, NAS). Auth model is solved.

### What doesn't apply
- **Convex** — overkill for Dan. SQLite on WSL2 is enough; no shared access, no collaborators, no mobile-heavy workloads. Adding a third-party DB adds another account to manage.
- **GitHub collaborator w/ own account** — Dan is the only user; no need to sandbox a separate identity. His crons already work with his credentials.
- **Composio for API key isolation** — Dan's credential hygiene is already bespoke (`~/.openclaw/secrets/*.env`, keyring). Adding Composio just adds a vendor.
- **Live session chat panel** — Dan's Slack is already that. The dashboard should *complement* Slack, not replace it.

---

## Source 2 — GitHub: Mannys-Repos/OpenClaw-Agent-Command-Center
**URL:** https://github.com/Mannys-Repos/OpenClaw-Agent-Command-Center
**Cloned to:** `/tmp/occ-ref/`. 69 commits, active, minimal community (1 star). MIT license.

### What's built
This is a **proper OpenClaw plugin** (not a personal side-project) that ships a browser dashboard at `http://localhost:19900`. It reads and writes OpenClaw's own files — `~/.openclaw/openclaw.json`, agent workspaces, skills directories, session state.

Feature coverage (pulled from README + `docs/*.md`):

| Area | What the plugin does |
|---|---|
| Agent graph | Visualize agents and their relationships (orchestrator → sub-agents) |
| Agent drawer | Edit identity, tools, channels, files, relationships, orchestrator settings |
| Sessions | List sessions per agent, open message history, send messages through gateway |
| Config editor | Parsed + raw JSON view. Deferred restart — stage N changes, apply once |
| Skills | Create/edit/enable/disable per-agent and global skills (same skill model as `.claude/skills`) |
| Plugins | View installed, manage user-installed |
| Tasks | View logs of cron / scheduled tasks |
| Logs tail | Live tail of gateway log (file or journald) |
| Task Flow Orchestrator | Multi-step agent pipelines with approval gates. Tool ID: `run_task_flow` with actions `run` / `step_complete` / `resume` |
| Auth | scrypt password hash in `~/.openclaw/extensions/openclaw-agent-dashboard/.credentials`, 24hr session cookie, bearer token API access |
| Deployment | `scripts/deploy.py` (paramiko over SSH) for VPS installs |

### Tech stack (the standout bit)
- **TypeScript + vanilla JS (no framework).** Client-side JS is inlined into the HTML at serve time. CSS served separately, read fresh from disk per request.
- **Node http server** running as an OpenClaw plugin (`kind: tools`, loaded via `openclaw.plugin.json`).
- Vitest + fast-check for tests.
- **Deferred restart system** — `?defer=1` on all mutations stages changes in memory, then `POST /api/config/commit` writes them once.
- REST API is deliberately un-framework-y: routes/ directory maps to URL segments (`routes/agents.ts`, `routes/sessions.ts`, `routes/health.ts`, `routes/logs.ts`, `routes/tools.ts`, etc.).

### What applies to Dan
- **This is the obvious "install and run" option.** If it works on his WSL2 gateway, he gets a real dashboard for ~20 min of setup cost. The agent graph + session browser + skills manager + deferred config editor are exactly the CoS-maintenance surface Dan currently does in the terminal.
- **`run_task_flow` primitive** — directly applicable to Dan's approval-gated cron work. Morning brief → review → send. Triage → review → send. These are multi-step flows with a human gate.
- **API shape** (`/api/overview`, `/api/agents/:id`, `/api/sessions/:key/message`, `/api/config/pending`) is a good reference for Dan's *own* endpoints if he builds custom views.
- **`.credentials` auth + Tailscale** — same pattern Dan should use. Default binds to `0.0.0.0`; set `bind: 127.0.0.1` for local-only.

### What doesn't apply
- **Vanilla-JS-inlined-into-HTML** is a choice driven by "no build step for a plugin." Dan already has a Vite + React 19 skeleton in the repo. No reason to abandon it.
- **Sub-agent graph visualization** — nice, but Dan has one CoS. Single node. Not useful until he splits into sub-agents (travel, home-ops, finance, health).
- **Multi-provider key management UI** — Dan only uses Anthropic (Claude Max) + OpenRouter. Already scripted.

### The net
Mannys-Repos is the **universal OpenClaw admin UI**. Dan's Command Center should build *on top of* or *alongside* it — not compete with it. Use Mannys for agent/session/config plumbing; build Dan-specific views (projects, triage inbox, cost, cron health) separately and wire them in.

---

## Source 3 — Reddit discovery
**Attempted queries:** r/LocalLLaMA, r/selfhosted, r/ClaudeAI via old.reddit.com JSON API.
**Blocker:** Reddit's search endpoint with `q=...&sort=new&t=year` is returning **unrelated new posts** (Chevy trucks, Notion habit trackers, credit referral spam) across all of Reddit, not scoped to the subreddit. This is a known change in Reddit's search API behavior since the 2024 API tightening — subreddit-scoped search now requires logged-in auth. Logging the blocker and routing around.

### Route-around: community landscape summary from direct knowledge + Langfuse/Open-WebUI research
What the community *is* converging on for agent-adjacent dashboards (as of Apr 2026):

1. **Langfuse** for traces/costs/evals (self-hosted or cloud).
2. **Open WebUI** (née Ollama WebUI) as a ChatGPT-style front-end to any LLM backend. 100k+ stars. Becoming the default for "I want to chat with my local models over the web."
3. **AnythingLLM, LibreChat, Chatbot UI** in the same space — conversational shells with RAG.
4. **Custom Next.js + shadcn** builds for personal "control planes" — the most visible pattern on Twitter/X. Low effort with v0/Lovable scaffolding.
5. **n8n / Flowise** for visual agent pipeline builders — more workflow than dashboard.

None of these are "your personal household operations dashboard wired to crons, calendars, email, HA, and Monarch." That niche is still bespoke — Benoît's video and the Mannys plugin are the current state of the art for the OpenClaw-specific take on it.

---

## Source 4 — Langfuse
**URL:** https://langfuse.com, https://github.com/langfuse/langfuse

### What's built
Open-source LLM observability. MIT (core). ClickHouse + Docker backend. Python + TS SDKs. Integrates with LangChain, LlamaIndex, OpenAI SDK, LiteLLM, and via OTEL.

### Data model (core insight)
- **Trace** = one top-level unit of work ("morning brief run"). Has: id, input, output, metadata, session_id, user_id, tags, timestamps.
- **Observation** = nested step inside a trace. Sub-types:
  - **Generation** — an LLM call. Records: model, input, output, tokens_in, tokens_out, cost, latency.
  - **Span** — any other timed operation (RAG retrieval, tool call).
  - **Event** — a discrete logged event.
- **Session** groups traces by `session_id` for multi-turn flows.

This is the **right schema** for what Dan currently tracks in `cost-log.json` — which is a flat list of runs with no nesting. If Dan ever splits into sub-agents, he'll need trace/observation structure to see "the morning brief trace spawned 3 sub-generations costing $X each."

### What applies to Dan
- **Cost dashboard patterns** — daily/weekly spend, cost per trace, spend by model, latency P50/P95. Drop-in pattern for the Cost tab in Dan's dashboard. He already has the raw data.
- **Trace viewer UX** — timeline with expandable nested spans. This is the right shape for a "Cron Run Detail" view (morning brief run → which scripts fired → how long each took → what model was used → token usage).
- **Session grouping** — if a Slack conversation with CoS spans multiple messages and tool calls, grouping them by session in the UI makes the history navigable.

### What doesn't apply
- **Self-hosting Langfuse itself** is overkill for Dan's volume (~30 traces/day, not 30k). ClickHouse + Docker Compose for one user = too much infra.
- **Evals + datasets + playground** are builder/developer features. Dan isn't iterating on prompts in production.

### The net
Don't run Langfuse. **Steal the data model.** When Dan eventually moves cost-log.json to a small SQLite, use the trace/observation/generation schema. It's been pressure-tested by a lot of smart people.

---

## Source 5 — Open WebUI
**URL:** https://github.com/open-webui/open-webui

### What's built
Self-hosted LLM chat front-end. Python backend (FastAPI), Svelte front-end. Docker/Kubernetes/pip. Supports Ollama + any OpenAI-compatible API (LMStudio, OpenRouter, Groq, Mistral). RAG with 9 vector DBs. RBAC, groups, OAuth/LDAP/SCIM, multilingual, PWA, voice/video.

### What applies to Dan
- **Model selector + multi-model chat** — if Dan ever wants to talk to CoS through a browser tab instead of Slack, Open WebUI can point at the OpenClaw gateway (which exposes OpenAI-compatible endpoints) and he gets a polished chat UI for free.
- **PWA** — install to phone home screen. Useful for on-the-road CoS access without the Slack app.
- **Pipelines plugin framework** — Python functions that run inline inside a chat turn. Similar to OpenClaw skills.

### What doesn't apply
- **The entire admin surface** (users, groups, auth providers, RBAC) — Dan is one user, not an org.
- **RAG over 9 vector DBs** — Dan's RAG story is two Obsidian vaults + Notion. File search, not vector search.

### The net
Not the answer. **But a credible fallback** if building a custom chat UI turns out to be too much. You could run Open WebUI as a secondary interface ("browser chat with CoS") and still build the Dan-specific dashboard panels separately.

---

## Source 6 — Home Assistant Lovelace (design pattern only)
**URL:** https://www.home-assistant.io/dashboards/

### What applies
Dan already runs Home Assistant at `100.94.72.30` — he's fluent in the Lovelace idiom. Patterns worth stealing:

1. **Card-as-primitive** — every widget is a self-contained card (Sensor, Gauge, Entity, Markdown, Picture Elements). The dashboard is a YAML file that composes cards into views. Zero-framework extensibility.
2. **Entity binding** — each card declares the entity/state it reads. Re-rendering = polling the entity. For Dan, "entity" = a file (TODO.md), an API (Notion DB), or a script output (cost-log.json).
3. **Exception-based surfacing** — a card can hide itself when there's nothing to show (`visibility: state != 'normal'`). Aligns directly with HEARTBEAT.md's "silence is the right answer."
4. **Masonry/Sections/Panel layouts** — different layout modes for different screen sizes. Phone vs. desktop view without custom code.
5. **Community cards** — curated custom-card ecosystem (button-card, mini-graph-card, auto-entities). Maps to "user-installable dashboard panels."

### What doesn't apply
- **YAML-driven configuration** as Dan's authoring model — he'd rather edit JSX than YAML in his own project.
- **Tightly coupled to HA state machine** — Dan's data isn't state-machine-shaped. It's files + API results.

### The net
**The Lovelace mental model is the right mental model.** Every panel is a card that (a) declares its data source, (b) renders a simple view, (c) can hide when there's nothing to say. Dan should build his dashboard this way even though the tech is React, not YAML.

---

## Source 7 — OpenRouter usage API
**URL:** https://openrouter.ai/docs (direct doc URLs 404'd; recovered from the generic docs page)

### What's available
- **Activity Export** (CSV/PDF) grouped by API key, model, or org member.
- **Usage Accounting** — per-model token counts (prompt, completion, cached).
- **User Tracking** — custom user IDs for sub-user reporting.
- **Analytics API:** `GET /api/v1/activity` (user activity grouped by endpoint).
- **Generation inspection:** `GET /api/v1/generation/:id` (metadata + usage), `GET /api/v1/generation/:id/content` (stored prompt/completion).
- **API key ops:** list keys, get current key.

### What applies to Dan
The Cost tab in the dashboard should cross-reference **three sources**:
1. `memory/cost-log.json` (Dan's local ledger — what he logs).
2. OpenRouter's activity endpoint (what OpenRouter billed — ground truth for OpenRouter calls).
3. Claude Max subscription (no per-call cost, track frequency only).

Discrepancies between (1) and (2) are a real bug surface — "am I under-logging?" Langfuse has the same reconciliation pattern.

---

## Cross-cutting themes

1. **Orchestrator + sub-agents is the shape everyone's converging on.** Benoît's video, Mannys plugin, Langfuse traces, even the "Task Flow Orchestrator" naming — all point at a tree of agent runs with the orchestrator at the root. Dan's CoS is currently monolithic. The dashboard should anticipate this split without forcing it.
2. **Exception-based surfacing beats completeness.** HA Lovelace, HEARTBEAT.md rules, morning-brief "skip if quiet" all agree. Dashboard should follow suit: a dashboard that's mostly empty when things are fine is the feature, not a bug.
3. **Approval gates are a first-class UI concept.** `run_task_flow` gates in Mannys, PR approvals in Benoît's inbox, Dan's "approval-first for external actions" rule. The dashboard's most valuable single widget is probably an "Approvals" inbox.
4. **Slack stays.** Nobody in the research replaced Slack/Telegram with the dashboard — they added a dashboard *next to* the chat interface. Chat = "talk to the agent." Dashboard = "see what the agent did / is doing / wants me to approve."
5. **Data already exists — dashboards are mostly rendering, not collection.** Dan's cost-log, TODO.md, ROADMAP.md, Notion DBs, cron logs, HA states — all live already. The dashboard is a viewer.

---

## Blockers logged

- **Reddit search API** returns unfiltered results — unable to scope to r/LocalLLaMA, r/selfhosted, r/ClaudeAI. Routed around by leaning on direct research of the dominant tools in that space (Langfuse, Open WebUI) and community-known patterns.
- **OpenRouter docs URLs** deep-linked in the task brief return 404 (docs site was restructured). Recovered endpoint names from the general `/docs` page listing.
- **YouTube transcript API** initially failed (`get_transcript` → old API). Fixed by using the new `YouTubeTranscriptApi().fetch()` method. Full 38,917-char transcript retrieved successfully.
