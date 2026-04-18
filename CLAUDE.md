# CLAUDE.md — Life Brain

> Read this file at the start of every session in this vault. It is the canonical orientation.

## About Dan

- Lives in St. Louis, MO with partner **Andrea**, two young kids, and a dog named **Blue**
- Owns primary residence (active renovations: retaining walls, kitchen, planned bathroom & basement)
- Owns and self-manages a rental property at **7025 Amherst Ave, University City, MO**
- Active fitness lifestyle, frequent work travel
- Hobbies: gardening (active sun-garden build), home improvement, AI tooling
- Systems-builder by nature — wants tools that compound, not tools that look good

## Purpose of this vault

This is Dan's **personal** second brain. It holds:
- Daily logs (personal side of the day)
- Life projects (garden, renovation, home improvements)
- Life areas (home, rental, fitness, family, finances, parenting)
- People — family, friends, neighbors, contractors, tenants
- Reference material (recipes, how-tos, manuals, research)
- Journal entries and reflections
- Travel planning and logs

**Not in this vault**: work content. Account intel, deal notes, Salesforce product research, and meeting notes with colleagues/customers live in the separate `work-brain` vault.

## Relationship to Notion

Notion remains the source of truth for:
- Structured databases (packing lists, recipes tracked relationally, shared household docs with Andrea)
- Anything requiring a table view

Obsidian = freeform thinking, narrative notes, journal, project logs. When a page exists in both, record the Notion URL as `notion_url:` in frontmatter.

## Relationship to OpenClaw Home CoS

Dan's "Home CoS" agent (defined by `IDENTITY.md`, `BOOTSTRAP.md`, `USER.md` in the OpenClaw workspace) will frequently read and write this vault. The Slack channel is the primary capture surface on mobile — messages there often should land as inbox notes or daily note additions here.

## Folder structure — flat by design

```
00-Inbox/       Capture zone. Unprocessed notes land here. Process weekly.
10-Daily/       Daily notes. Nested by year: 10-Daily/2026/2026-04-17.md
20-People/      Family, friends, contractors, tenants. One file per person.
30-Projects/    Active projects with a defined outcome (e.g., "Sun Garden Build").
40-Areas/       Ongoing responsibilities with no end date (Home, Rental, Fitness, Family, Finances).
50-Resources/   Reference material — guides, recipes, research, how-tos.
60-Journal/     Personal reflections, weekly reviews, longer-form writing.
70-Travel/      Trip planning, itineraries, travel logs.
80-MOCs/        Maps of Content — topic indexes. Primary AI entry points.
90-Archive/     Completed projects, inactive areas.
_templates/     Templater templates. Do not write user content here.
_claude/        AI scratch — drafts, summaries, generated artifacts. Quarantine zone.
_attachments/   Images, PDFs, screenshots.
```

**Hard rules**:
- Do not create new top-level folders.
- Projects have outcomes and end dates. Areas don't. If it's not clear, ask Dan.
- Do not put AI drafts in content folders. Quarantine in `_claude/`.

## YAML frontmatter — canonical schema

Every note MUST have frontmatter. Required fields: `type`, `status`, `created`.

```yaml
---
type: daily | project | area | person | reference | journal | travel | moc | fleeting
status: active | stale | archived | idea | done
created: 2026-04-17
updated: 2026-04-17
tags: []
---
```

Type-specific extensions:

**project**:
```yaml
type: project
outcome: "Brief description of what 'done' looks like"
target_date: 2026-06-01
area: "[[Home]]"
```

**area**:
```yaml
type: area
review_frequency: weekly | monthly | quarterly
```

**person**:
```yaml
type: person
relationship: family | friend | contractor | tenant | neighbor | other
phone:
email:
```

**travel**:
```yaml
type: travel
destination: Chicago
start_date: 2026-05-10
end_date: 2026-05-12
with: ["[[Andrea]]", "[[kids]]"]
```

## Naming conventions

- **Files**: Title Case, descriptive, <60 characters.
- **Daily notes**: `YYYY-MM-DD.md` (ISO date).
- **MOCs**: `<Topic> MOC.md` — suffix required.
- **Wikilinks**: `[[File Name]]`. Prefer linking existing notes.
- **Tags**: lowercase, kebab-case. `#home-reno` not `#HomeReno`.

## Rules for AI-generated content

1. Drafts go in `_claude/` with dated filenames like `_claude/2026-04-17-garden-planning-options.md`.
2. Dan promotes manually to canonical folders. No self-promotion.
3. When editing existing notes, preserve Dan's prose. Fence new AI sections with `<!-- claude: start -->` / `<!-- claude: end -->`.
4. Never modify `_templates/`.
5. Never touch `90-Archive/` without explicit instruction.

## Standard workflows

### Daily kickoff (personal side of the morning)
1. Create today's note at `10-Daily/YYYY/YYYY-MM-DD.md` from template if missing.
2. Carry over unfinished personal tasks from yesterday.
3. Surface anything scheduled today from active projects.

### Weekly review (Sunday)
1. Read all daily notes from the week.
2. Write review to `60-Journal/YYYY-WW.md` covering: wins, lessons, what mattered, what's ahead.
3. Flag projects not touched in 14+ days as `status: stale`.
4. Tidy `00-Inbox/`.

### Home / Rental logging
When Dan logs something about the house or rental (a fix, a vendor call, a receipt):
1. Add an entry to the relevant area note (`40-Areas/Home.md` or `40-Areas/Rental Property 7025 Amherst.md`).
2. Link the vendor as a `[[Person]]` if they'll likely come up again.
3. Drop receipts/photos in `_attachments/` and link.

### Garden logging
- Log into `80-MOCs/Garden MOC.md` or the active `30-Projects/Sun Garden Build.md`.
- Plant entries: one line per event with date prefix (planting, bloom, issue, dividing).

## What NOT to do

- Don't add frontmatter fields outside this schema without asking.
- Don't rename files without updating inbound wikilinks.
- Don't merge multiple concerns into one note — split by concept, link by wikilink.
- Don't treat journal entries as editable — they are authentic thinking.
- Don't schedule or advise on infant sleep, health, or parenting without Dan confirming he wants AI input on that topic in this session.
