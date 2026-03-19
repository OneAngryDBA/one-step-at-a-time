# Areas of Responsibility — Design Specification

**Version:** 2.0.0
**Status:** Implemented (Phase 1 — hierarchy support)

---

## Overview

Areas represent ongoing responsibilities in your life — things with no end date that require regular attention. Based on the PARA method (Projects, Areas, Resources, Archives), Areas sit alongside Projects as an organizational layer, giving structure to habits, chores, maintenance, and obligations.

**v4.0 additions:** Path-based area hierarchy (sub-areas), template references on recurring items, integration with the persistent task store.

---

## Core Concepts

### Areas vs Projects

| | Areas | Projects |
|---|---|---|
| **End state** | None — ongoing | Has a goal + deadline |
| **Examples** | Home, Health, Finances | Kitchen Renovation, Tax Filing 2026 |
| **Tag prefix** | `@` (e.g., @home, @work/engineering) | `#` (e.g., #kitchen-reno) |
| **Contains** | Recurring items | Tasks (in task store) |
| **Storage** | `AREAS.md` | `PROJECTS.md` |
| **Hierarchy** | Path-based sub-areas | Flat |

### Relationship Rules

- A **task** belongs to EITHER a project OR an area, never both
- A **project** can optionally belong to an area (e.g., #kitchen-reno → @home)
- A **recurring item** can be promoted to a project when it becomes complex
- A **recurring item** can reference a template for automatic task creation

### Three Item Tiers

**🧘 Habits** — Daily routines, always soft, no consequences
- Brush teeth, eat meals, exercise, meditate
- Tracked by frequency (daily, X times per week)

**🔄 Maintenance** — Periodic chores, soft deadlines, no formal consequences
- Vacuum, mow lawn, clean gutters, oil change
- Tracked by cadence (weekly, monthly, seasonal)

**📋 Obligations** — Hard deadlines with consequences that escalate
- File taxes, pay bills, renew license, insurance payments
- Tracked by hard due dates with documented consequences

---

## Area Hierarchy

### Path-based tags

Areas support unlimited depth in the data model, with Phase 1 implementing one level:

| Tag | Type | Example |
|-----|------|---------|
| `@work` | Top-level area | Work responsibilities |
| `@work/engineering` | Sub-area | Engineering-specific work |
| `@work/management` | Sub-area | Management-specific work |
| `@home` | Top-level area | Home responsibilities |
| `@home/garden` | Sub-area | Garden maintenance |

### Hierarchy rules

- Path separator is `/`
- Parent areas can have their own recurring items
- Sub-areas inherit parent context but have their own items and counts
- Tasks tagged `@work/engineering` are scoped to that sub-area
- Tasks tagged `@work` apply to any work context
- In Phase 2, energy blocks can be area-scoped (a `🧠 Deep @work` block can be used by `@work/engineering` tasks)

### Configuration

```json
"areas": {
  "hierarchyEnabled": true,
  "maxDepth": 2
}
```

`maxDepth: 2` means top-level + one sub-level. The data model supports deeper nesting for future use.

---

## Storage Format

### AREAS.md Structure (with hierarchy)

```markdown
# Areas of Responsibility

## Work @work
**Items:** 8 | **Projects:** 3 | **Sub-areas:** 2

**Recurring:**
- 📋 Weekly status report — Every week (Fri) — Last: 2026-03-14 — ✅ Next: 2026-03-21

### Engineering @work/engineering
**Items:** 3 | **Projects:** 2

**Recurring:**
- 🔄 Review PRs — Every day (weekdays) — Last: 2026-03-18 — ✅ On track
- 📋 Sprint retrospective — Every 2 weeks (Fri) — Last: 2026-03-07 — 📅 Due soon
  Template: sprint-retro

### Management @work/management
**Items:** 2 | **Projects:** 1

**Recurring:**
- 📋 1:1 meetings — Every week (Tue) — Last: 2026-03-18 — ✅ On track

**Projects:** #api-redesign, #docs-update

---

## Home @home
**Items:** 5 | **Projects:** 2 | **Sub-areas:** 2

**Recurring:**
- 🧘 Make bed — Daily — Last: 2026-03-14 — ✅ On track
- 📋 Pay mortgage — Every month (1st) — Last: 2026-03-01 — ✅ Next: 2026-04-01
  ⚠️ If missed: Late fee ($50). After 30 days: credit score impact

### Maintenance @home/maintenance
**Items:** 3

**Recurring:**
- 🔄 Vacuum house — Every week (Sat) — Last: 2026-03-08 — ⚠️ Due today
- 🔄 Mow lawn — Every 2 weeks (seasonal: Apr-Oct) — Last: n/a — 💤 Off-season
- 📋 Change HVAC filter — Every 3 months — Last: 2026-01-10 — 🔴 Overdue (5 days)
  ⚠️ If missed: Reduced efficiency, potential system damage

### Garden @home/garden
**Items:** 2

**Recurring:**
- 🔄 Water plants — Every 3 days — Last: 2026-03-17 — ✅ On track
- 🔄 Prune hedges — Every month (seasonal: Mar-Oct) — Last: 2026-02-28 — 📅 Due soon

**Projects:** #kitchen-reno, #garage-org

---

## Health @health
**Items:** 3 | **Projects:** 1

**Recurring:**
- 🧘 Exercise — 4x/week — This week: 2/4 — ✅ On track
- 🔄 Annual physical — Every year — Last: 2025-06-15 — ✅ Next: 2026-06-15
- 📋 Dentist cleaning — Every 6 months — Last: 2025-11-01 — 📅 Due soon (2026-05-01)
  ⚠️ If missed: Potential cavities, gum disease progression

**Projects:** #couch-to-5k

---
```

### Key format points

- Top-level areas use `## Name @tag`
- Sub-areas use `### Name @parent/child-tag` (nested inside parent section)
- Parent areas show `**Sub-areas:** N` in their header
- The `**Projects:**` line appears at the end of the top-level area section (after sub-areas)
- Sub-areas do NOT have their own Projects line — projects link to the top-level area or sub-area via `**Area:** @tag` in PROJECTS.md

### Recurring Item Format

**🧘 Habits:**
```
- 🧘 {{description}} — {{cadence}} — Last: {{YYYY-MM-DD}} — {{status}}
```

**🔄 Maintenance:**
```
- 🔄 {{description}} — {{cadence}} — Last: {{YYYY-MM-DD}} — {{status}}
```

**📋 Obligations:**
```
- 📋 {{description}} — {{cadence}} — Last: {{YYYY-MM-DD}} — {{status}}
  ⚠️ If missed: {{consequence}}. After {{time}}: {{escalation}}
```

**With template reference:**
```
- 📋 {{description}} — {{cadence}} — Last: {{YYYY-MM-DD}} — {{status}}
  Template: {{template-name}}
  Lead: {{Nd}}
```

### Cadence Formats

| Cadence | Format | Example |
|---------|--------|---------|
| Daily | `Daily` | Brush teeth |
| Specific days | `Every Mon, Wed, Fri` | Exercise |
| X times per week | `4x/week` | Exercise |
| Weekly on day | `Every week (Sat)` | Vacuum |
| Every N days | `Every 3 days` | Water plants |
| Every N weeks | `Every 2 weeks` | Mow lawn |
| Monthly on date | `Every month (1st)` | Pay rent |
| Every N months | `Every 3 months` | HVAC filter |
| Yearly on date | `Every year (Apr 15)` | File taxes |
| Seasonal | `Every 2 weeks (seasonal: Apr-Oct)` | Mow lawn |

### Status Values (Auto-Calculated)

| Status | Meaning | Applies to |
|--------|---------|------------|
| `✅ On track` | Next due date is in the future | All |
| `✅ Next: YYYY-MM-DD` | On track, showing next due | All |
| `📅 Due soon` | Within configurable window (default 2 days) | All |
| `⚠️ Due today` | Due today | All |
| `🔴 Overdue (N days)` | Past due | 📋 Obligations |
| `🟡 Nudge` | Past cadence (gentle) | 🧘 Habits, 🔄 Maintenance |
| `💤 Off-season` | Seasonal item outside active window | All |
| `⏸️ Paused` | Manually paused by user | All |

---

## Template Integration

Recurring items can reference templates from `templates/tasks/`:

```markdown
- 📋 Monthly financial close — Every month (last business day) — Last: 2026-02-28
  Template: monthly-close
  Lead: 3d
```

**How it works:**
1. At `/start-day`, when the item is due (or within Lead time), the system suggests: "📋 Monthly close is due — instantiate template?"
2. User confirms → `/new-from-template monthly-close` runs
3. Template creates tasks in the task store with the area's @tag
4. User selects which tasks to focus on today

This bridges the gap between "recurring item is due" and "I have concrete tasks to work on."

---

## Item → Project Promotion

Some recurring items become complex enough to warrant a project:
- "File taxes" → create project with subtasks
- "Annual physical" → might spawn project if follow-up care is needed

**How it works (via `/update-area`):**
1. Select "Promote to project"
2. Choose the recurring item
3. System creates a new project linked to the area (or sub-area)
4. The recurring item stays in the area (it'll recur next cycle)
5. The project gets `**Area:** @tag` automatically
6. If a template exists, suggest instantiation for task creation

---

## Tag System

### Format
- Areas: `@area-tag` or `@area/sub-area` (lowercase, hyphen-separated)
- Projects: `#project-tag` (existing format)

### Mutual Exclusivity
- A task has at most ONE tag — either `@area` or `#project`
- If a task belongs to a project, the area context comes from the project's `**Area:** @tag` field
- Commands enforce this at link time

### Sub-area scoping
- A task tagged `@work/engineering` belongs specifically to the engineering sub-area
- A task tagged `@work` belongs to the broader work area
- In Phase 2, energy blocks scoped to `@work` can be used by `@work/engineering` tasks (child inherits parent's blocks)

---

## Load Check

The system measures load without making emotional assumptions.

### Metrics (computed at `/start-day`)
1. Today's task count (from task store in-progress)
2. Today's total time estimate (sum of 🔧)
3. Recurring items due/nudging
4. Overdue obligations count

### Thresholds (configurable)
```json
"areas": {
  "loadCheck": true,
  "loadThresholds": {
    "dailyTaskCount": 8,
    "dailyTimeMinutes": 360,
    "overdueRecurringItems": 3,
    "totalDueRecurringItems": 5
  }
}
```

### Display (when thresholds exceeded)
```
📊 Load Check:
- 12 tasks in focus (threshold: 8)
- 6 recurring items due
- 3 obligations overdue

Options: Move tasks back to ready / Triage recurring / Proceed as-is
```

---

## Commands Summary

### Area commands
- `/new-area` — Create a new area or sub-area with recurring items
- `/list-areas` — Show all areas with status (including hierarchy)
- `/update-area` — Manage items, sub-areas, promote to project
- `/review-areas` — Periodic area review

### Modified commands (v4.0)
- `/start-day` — Area reminders with template suggestions + lead time
- `/review-day` — Update recurring item Last dates from task store completions
- `/dump` / `/brain-dump` / `/add-task` — Area linking with sub-area support
- `/link-task` — Support area tags including sub-areas
- `/new-project` — Area linking with sub-area support

---

## Configuration

```json
"areas": {
  "enabled": true,
  "hierarchyEnabled": true,
  "maxDepth": 2,
  "showInStartDay": true,
  "showInReviewDay": true,
  "dueSoonDays": 2,
  "loadCheck": true,
  "loadThresholds": {
    "dailyTaskCount": 8,
    "dailyTimeMinutes": 360,
    "overdueRecurringItems": 3,
    "totalDueRecurringItems": 5
  }
}
```

---

## Philosophy

Consistent with OStaaT principles:
- **Reduce friction** — easy to add and track recurring items, templates automate setup
- **Create clarity** — three tiers + hierarchy make it obvious what needs attention
- **Bias toward action** — lead times and templates bridge "item is due" to "here are tasks"
- **Never assume emotions** — present data, let the user decide
- **Don't invent** — only track what the user explicitly adds
- **Structure when needed** — sub-areas add organization without forcing hierarchy
