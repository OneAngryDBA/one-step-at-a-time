# Areas of Responsibility — Design Specification

**Version:** 1.0.0
**Status:** Design Complete — Implementation In Progress

---

## Overview

Areas represent ongoing responsibilities in your life — things with no end date that require regular attention. Based on the PARA method (Projects, Areas, Resources, Archives), Areas sit alongside Projects as an organizational layer, giving structure to habits, chores, maintenance, and obligations.

---

## Core Concepts

### Areas vs Projects

| | Areas | Projects |
|---|---|---|
| **End state** | None — ongoing | Has a goal + deadline |
| **Examples** | Home, Health, Finances | Kitchen Renovation, Tax Filing 2026 |
| **Tag prefix** | `@` (e.g., @home) | `#` (e.g., #kitchen-reno) |
| **Contains** | Recurring items | Tasks |
| **Storage** | `AREAS.md` | `PROJECTS.md` |

### Relationship Rules

- A **task** belongs to EITHER a project OR an area, never both
- A **project** can optionally belong to an area (e.g., #kitchen-reno → @home)
- A **recurring item** can be promoted to a project when it becomes complex

### Three Item Tiers

Areas contain recurring items in three tiers:

**🧘 Habits** — Daily routines, always soft, no consequences
- Brush teeth, eat meals, exercise, meditate
- Tracked by frequency (daily, X times per week)
- Missing one is fine — gentle nudge at most

**🔄 Maintenance** — Periodic chores, soft deadlines, no formal consequences
- Vacuum, mow lawn, clean gutters, oil change
- Tracked by cadence (weekly, monthly, seasonal)
- Missing one isn't critical but builds up over time

**📋 Obligations** — Hard deadlines with consequences that escalate
- File taxes, pay bills, renew license, insurance payments
- Tracked by hard due dates
- Missing one has real consequences (fines, penalties, damage)
- Consequences documented and surfaced when overdue

---

## Storage Format

### AREAS.md Structure

```markdown
# Areas of Responsibility

## Home
**Tag:** @home | **Items:** 5 | **Projects:** 2

**Recurring:**
- 🧘 Make bed — Daily — Last: 2026-03-14 — ✅ On track
- 🔄 Vacuum house — Every week (Sat) — Last: 2026-03-08 — ⚠️ Due today
- 🔄 Mow lawn — Every 2 weeks (seasonal: Apr-Oct) — Last: n/a — 💤 Off-season
- 📋 Change HVAC filter — Every 3 months — Last: 2026-01-10 — 🔴 Overdue (5 days)
  ⚠️ If missed: Reduced efficiency, potential system damage
- 📋 Pay mortgage — Every month (1st) — Last: 2026-03-01 — ✅ Next: 2026-04-01
  ⚠️ If missed: Late fee ($50). After 30 days: credit score impact. After 90 days: foreclosure proceedings

**Projects:** #kitchen-reno, #garage-org

---

## Health
**Tag:** @health | **Items:** 3 | **Projects:** 1

**Recurring:**
- 🧘 Exercise — 4x/week — This week: 2/4 — ✅ On track
- 🔄 Annual physical — Every year — Last: 2025-06-15 — ✅ Next: 2026-06-15
- 📋 Dentist cleaning — Every 6 months — Last: 2025-11-01 — 📅 Due soon (2026-05-01)
  ⚠️ If missed: Potential cavities, gum disease progression

**Projects:** #couch-to-5k

---
```

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

For "X times per week" items, status includes progress: `This week: 2/4`

### Consequence Format (📋 Obligations only)

```
⚠️ If missed: {{immediate consequence}}. After {{time}}: {{escalation}}
```

- Written once when the item is created
- Surfaced in `/start-day` when the obligation is overdue
- Multiple escalation tiers supported (separated by periods)
- Examples:
  - `⚠️ If missed: Late fee ($29). After 30 days: interest + credit score impact`
  - `⚠️ If missed: Reduced air quality. After 6 months: potential system damage`
  - `⚠️ If missed: Late filing penalty + interest. After 60 days: failure-to-file penalty (5%/month)`

---

## Item → Project Promotion

Some recurring items become complex enough to warrant a project:
- "File taxes" → create project with subtasks (gather docs, fill forms, review, submit)
- "Annual physical" → might spawn project if follow-up care is needed

**How it works (via `/update-area`):**
1. Select "Promote to project"
2. Choose the recurring item
3. System creates a new project linked to the area
4. The recurring item stays in the area (it'll recur next cycle)
5. The project gets `**Area:** @tag` automatically

---

## Tag System

### Format
- Areas: `@area-tag` (lowercase, hyphen-separated)
- Projects: `#project-tag` (existing format)

### Mutual Exclusivity
- A task has at most ONE tag — either `@area` or `#project`
- If a task belongs to a project, the area context comes from the project's `**Area:** @tag` field
- Commands enforce this: if user picks a project, no area tag is added

### Project-Area Linkage
In PROJECTS.md, projects gain an optional area field:
```markdown
### Kitchen Renovation #kitchen-reno
**Status:** 🟢 Active | **Progress:** 40% | **Due:** 2026-06-01 | **Area:** @home
```

AREAS.md shows a cross-reference: `**Projects:** #kitchen-reno, #garage-org`

---

## Load Check

The system measures load without making emotional assumptions. It presents data and asks questions.

### Metrics (computed at `/start-day`)
1. Today's task count
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
- 12 tasks today (threshold: 8)
- 6 recurring items due
- 3 obligations overdue

Options: Triage recurring items / Move tasks to tomorrow / Proceed as-is
```

### "Things You'd Forget" (Gentle Reminders)
At `/start-day`, the system checks for:
- Items not done in significantly longer than their cadence (2x+ the period)
- Seasonal items entering their active window
- Items with "Due soon" status

```
🔔 Gentle Reminders:
- Change HVAC filter — last done 95 days ago (every 3 months)
- Mow lawn season starts next week
- Dentist cleaning due in 6 weeks — consider scheduling
```

---

## Commands Summary

### New Commands
- `/new-area` — Create a new area with recurring items
- `/list-areas` — Show all areas with status
- `/update-area` — Manage area items (add/edit/remove/pause/promote)
- `/review-areas` — Periodic area review

### Modified Commands
- `/start-day` — Add area reminders + load check
- `/review-day` — Add upcoming recurring items + update last-done
- `/dump` / `/brain-dump` — Add area linking option
- `/add-task` — Add area linking option
- `/link-task` — Support areas alongside projects
- `/new-project` — Ask about area membership
- `/review-projects` — Show area context

---

## Configuration

```json
"areas": {
  "enabled": true,
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

## Implementation Order

1. Foundation — AREAS.md, /new-area, /list-areas, config
2. Recurring Item Management — /update-area with status calculation
3. Daily Integration — /start-day + /review-day modifications
4. Task Linking — /dump, /add-task, /link-task modifications
5. Project-Area Integration — /new-project + /review-projects modifications
6. Polish — /review-areas, README update

---

## Philosophy

Consistent with Action Organizer principles:
- **Reduce friction** — easy to add and track recurring items
- **Create clarity** — three tiers make it obvious what needs attention
- **Bias toward action** — nudge, don't nag
- **Never assume emotions** — present data, let the user decide
- **Don't invent** — only track what the user explicitly adds
