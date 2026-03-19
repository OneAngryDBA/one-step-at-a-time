# Persistent Task Store — Design Specification

**Version:** 4.0.0
**Status:** Implementing (Phase 1)

---

## Overview

The persistent task store inverts OStaaT's data model. Previously, tasks lived only inside daily files (`YYYY-MM-DD-todo.md`) and "rolling forward" meant copying incomplete tasks from yesterday. Now, a sharded `tasks/` directory is the **source of truth**. Daily files become views — generated snapshots of what you're focusing on today.

### Problems solved

| Problem (v3) | Solution (v4) |
|---------------|---------------|
| Multi-day tasks endlessly carried over with `📌 Carried over from:` | Tasks persist in the store; `Focus-dates` tracks which days you worked on them |
| No global view of all open work | `INDEX.md` + status files give full visibility |
| No lifecycle states (just incomplete/complete) | Six states: inbox → ready → in-progress → waiting → paused → done |
| No reusable workflows | Unified templates create tasks + projects together |
| No capacity awareness | Energy model (Phase 2) with area-scoped calendars |

---

## Task Store Structure

**Location:** `{workspace}/{dataDir}/tasks/`

```
tasks/
├── INDEX.md         # t-NNN → status + one-line summary
├── inbox.md         # Untriaged captures
├── ready.md         # Actionable backlog
├── in-progress.md   # Currently focused
├── waiting.md       # Blocked on externals
└── paused.md        # Deliberately deferred
```

### Why sharded by status

- Commands only read the files they need (`/start-day` reads `in-progress.md` + `ready.md` + `inbox.md`)
- Moving a task between statuses = remove from one file, add to another
- Each file stays small (~20-80 tasks at scale of hundreds)
- Clean git diffs — only the affected status file changes
- Maps cleanly to database tables if/when we migrate to a real app

---

## Task IDs

- Format: `t-NNN` — short, sequential, zero-padded to 3 digits (expanding as needed)
- Examples: `t-001`, `t-042`, `t-117`
- **Never recycled** — even after a task is done, that ID is retired
- Tracked via `tasks.nextId` in `ostaat.json`
- Square bracket format in files: `- [t-017] 🔴 High ...`
- Subtasks get their own IDs from the same sequence
- Done tasks keep their ID in INDEX.md (status = `done`) for historical reference

---

## INDEX.md — Lightweight Lookup Table

```markdown
# Task Index

| ID | Status | Summary | Tag |
|----|--------|---------|-----|
| t-001 | done | Set up CI pipeline | #infra |
| t-002 | ready | Schedule dentist | @health |
| t-017 | in-progress | API endpoint refactoring | #api-redesign |
| t-019 | waiting | Review accountant response | #tax-filing |
| t-022 | ready | Dentist appointment | @health |
| t-025 | inbox | Look into monitoring tool | — |
| t-030 | ready | Monthly financial close | #monthly-close |
```

### Purpose

- Fast lookups without parsing all status files
- One-line-per-task summary for commands like `/list-tasks`
- Historical record — done tasks stay in the index
- Updated atomically with every task operation

---

## Task Format

### Full format

```markdown
- [t-NNN] 🔴|🟡|🟢 Priority ⏰|⏭️|📅 Timing 🔧 Nmin [🕓 Nmin] [EnergyEmoji EnergyLabel] [📅 YYYY-MM-DD] [#project|@area/sub]
  Description text
  [Dependencies: t-NNN, t-NNN]
  [Context: ...]
  [Lead: Nd]
  [Start: YYYY-MM-DD]
  [Waiting-on: ...]
  [Waiting-since: YYYY-MM-DD]
  [Expected: YYYY-MM-DD]
  [Paused-reason: ...]
  [Created: YYYY-MM-DD]
  [Focus-dates: YYYY-MM-DD, ...]
```

### Field reference

| Field | Required | Notes |
|-------|----------|-------|
| ID `[t-NNN]` | Yes | Unique, sequential, never recycled |
| Priority | Yes | 🔴 High, 🟡 Medium, 🟢 Low |
| Timing | Yes | ⏰ Now, ⏭️ Next, 📅 Later (urgency within lifecycle state) |
| Active time `🔧` | Yes | Hands-on work time in minutes |
| Passive time `🕓` | No | Waiting/processing time |
| Energy label | No (Phase 2 enforces) | User-defined, e.g., `🧠 Deep`, `⚡ Admin` |
| Due date `📅` | No | Hard deadline |
| Tag | No | `#project` or `@area/sub` (mutually exclusive) |
| Description | Yes | Second line, indented |
| Dependencies | No | Task IDs this depends on |
| Context | No | Links, notes, resources |
| Lead | No | `Nd` — surface N days before due date |
| Start | No | Explicit start date (overrides Lead) |
| Waiting-on | No | What/who this task is blocked by (for waiting status) |
| Waiting-since | No | When task entered waiting state |
| Expected | No | When the blocker is expected to resolve |
| Paused-reason | No | Why this task was deliberately deferred |
| Created | Yes (auto) | Date task was created |
| Focus-dates | Auto | Days this task was actively worked on |

### Status vs Timing

These are orthogonal dimensions:

- **Status** = which file in `tasks/` the task lives in (lifecycle position)
- **Timing** (⏰/⏭️/📅) = urgency *within* a status

A task can be `ready` with timing `📅 Later` (in the backlog, not urgent) or `ready` with `⏰ Now` (should be picked up today).

---

## Task Lifecycle

```
inbox → ready → in-progress → done
                    ↕
                 waiting
                    ↕
                  paused
```

### Status definitions

| Status | File | Meaning |
|--------|------|---------|
| inbox | `inbox.md` | Captured but not triaged — needs priority/timing/estimates |
| ready | `ready.md` | Triaged and actionable — available for daily focus selection |
| in-progress | `in-progress.md` | Selected for today's focus — actively being worked on |
| waiting | `waiting.md` | Blocked on an external dependency (person, event, approval) |
| paused | `paused.md` | Deliberately deferred — user chose to set this aside |
| done | (removed from status files) | Completed — INDEX.md updated, written to daily finished file |

### Transition rules

| From | To | Trigger |
|------|-----|---------|
| inbox | ready | User triages (assigns priority/timing) via `/update-task` or during `/start-day` |
| inbox | done | Quick completion — captured and done immediately |
| ready | in-progress | Selected during `/start-day` focus selection |
| in-progress | done | Marked complete in `/review-day` |
| in-progress | waiting | Blocked — user provides `Waiting-on` info |
| in-progress | paused | User deliberately defers |
| in-progress | ready | De-selected from today's focus |
| waiting | in-progress | Blocker resolved, user resumes |
| waiting | ready | Blocker resolved, not resuming immediately |
| paused | ready | User decides to resume |
| paused | in-progress | User decides to resume and work on it now |

### Done handling

When a task → `done`:
1. Remove from its current status file
2. Update INDEX.md: set status to `done`
3. Write task to today's `YYYY-MM-DD-finished.md`
4. If task has `@area-tag` and `🔄 Recurring` notation, update AREAS.md Last date

---

## Subtasks (Lightweight Nesting)

### Rules

- **Max 1 level deep** — no grandchildren
- Subtasks have their own `[t-NNN]` IDs from the same sequence
- Subtasks have their own time estimates and optional energy labels
- Subtasks inherit parent's project/area tag (don't repeat it)
- Subtasks can have dependencies on other subtasks or tasks
- Subtasks live in the same status file as their parent

### Parent format with subtasks

```markdown
- [t-030] 🔴 High ⏰ Now 🔧 120 min 🧠 Deep #monthly-close
  Monthly financial close (1/4 done)
  Due: 2026-03-31
  Lead: 3d
  Created: 2026-03-10
  - [t-031] 🔧 30 min 🕓 60 min ⚡ Admin
    Pull bank statements
  - [t-032] 🔧 30 min 🧠 Deep
    Reconcile accounts
  - [t-033] 🔧 20 min ⚡ Admin
    Generate reports
  - [t-034] 🔧 15 min ⚡ Admin
    Submit to accounting
    Dependencies: t-031, t-032, t-033
```

### Behavior

- Parent shows rollup: `(N/M done)` in description
- Parent active time `🔧` = sum of children's active times (auto-calculated)
- Subtasks can be independently selected for daily focus
- When a subtask is completed, it stays under the parent (marked done inline) until the parent completes
- When ALL subtasks are done → parent auto-completes
- In INDEX.md, subtasks are NOT listed separately — only the parent appears

---

## Status File Format

Each status file follows the same structure:

```markdown
# {{Status Name}}

- [t-NNN] 🔴 High ⏰ Now 🔧 45 min 📅 2026-03-20 #project-tag
  Task description
  Created: 2026-03-15

- [t-NNN] 🟡 Medium ⏭️ Next 🔧 30 min @health
  Another task
  Created: 2026-03-16
```

Tasks are ordered within each file by:
1. Priority (🔴 → 🟡 → 🟢)
2. Timing (⏰ → ⏭️ → 📅)
3. Due date (earliest first)
4. Created date (oldest first)

---

## Lead Time & Start Dates

Two mechanisms for surfacing tasks before their deadline:

### Lead time (`Lead: Nd`)

Relative. System calculates `start = due - lead`. Task highlighted at `/start-day` when `today ≥ due - lead`.

```markdown
- [t-030] 🔴 High ⏭️ Next 🔧 120 min #monthly-close
  Monthly financial close
  Due: 2026-03-31
  Lead: 3d
```

This task surfaces at `/start-day` starting March 28.

### Start date (`Start: YYYY-MM-DD`)

Absolute. If both Lead and Start are set, **Start wins**.

```markdown
- [t-040] 🟡 Medium 📅 Later 🔧 60 min
  Prepare quarterly review
  Due: 2026-04-15
  Start: 2026-04-01
```

### For area recurring items with templates

Lead on the recurring item determines when `/start-day` suggests template instantiation:
```markdown
- 📋 Monthly financial close — Every month (last business day)
  Template: monthly-close
  Lead: 3d
```

---

## Daily Files as Views

Daily files still exist but are **generated from the task store**.

### Generation flow (`/start-day`)

1. Read `in-progress.md` — show tasks from yesterday (pre-selected by default)
2. Read `ready.md` — show sorted by priority/due/lead time. Highlight where `Start ≤ today` or `Due - Lead ≤ today`
3. Read `waiting.md` — flag any where `Expected` date has passed
4. Read `inbox.md` — show for quick triage
5. Pull `scheduled/` tasks for today → assign IDs, write to `inbox.md` or `ready.md`
6. Show area reminders from AREAS.md. If recurring item has `Template:` field and is due, suggest instantiation
7. User confirms today's focus → daily file generated
8. Selected tasks moved to `in-progress.md` in store, today added to `Focus-dates`
9. Load check (unchanged)

### Daily file format

The daily file contains **full task copies** with IDs — self-contained and readable:

```markdown
# Thursday, March 19, 2026

## Tasks

- [t-017] 🔴 High ⏰ Now 🔧 45 min #api-redesign
  API endpoint refactoring
  Focus-dates: 2026-03-17, 2026-03-18, 2026-03-19

- [t-022] 🟡 Medium ⏭️ Next 🔧 30 min @health
  Schedule dentist appointment

## Completed

(tasks moved here during the day)
```

### Key difference from v3

- **No more rolling forward** — tasks persist in the store
- **`Focus-dates` replaces carry-over notes** — shows history without clutter
- **Task store is authoritative** — daily file is a working snapshot
- **IDs enable sync** — changes in daily file can be traced back to store

---

## `scheduled/` Directory (Kept)

- Remains for lightweight future items not yet worth tracking as persistent tasks
- When `/start-day` pulls scheduled tasks for today, they get **assigned IDs and written to the task store** (inbox or ready status)
- The scheduled file is then removed (current behavior)

---

## Configuration (`ostaat.json`)

New sections added in v4.0:

```json
{
  "tasks": {
    "enabled": true,
    "nextId": 1,
    "showInboxAtStartDay": true,
    "autoSelectInProgress": true,
    "storeDir": "tasks"
  },
  "energy": {
    "enabled": false,
    "dimensions": []
  },
  "templates": {
    "folder": "templates/tasks",
    "promptOnAreaDue": true
  },
  "areas": {
    "hierarchyEnabled": true,
    "maxDepth": 2
  }
}
```

### tasks.nextId

Tracks the next available task ID number. Incremented every time a task (or subtask) is created. Never decremented.

### tasks.storeDir

Directory name for the task store, relative to `{dataDir}`. Default: `"tasks"`.

---

## Migration Strategy

### v3 → v4 progression

1. **v4.0-alpha**: Add task store, dual-write to both task store and daily file
2. **v4.0-beta**: Commands read from task store as primary source, fall back to old behavior
3. **v4.0**: `/migrate-tasks` extracts tasks from existing daily files into task store
4. **v4.1**: Remove dual-write and backward compatibility

### `/migrate-tasks` command

One-time migration that:
1. Scans existing daily todo files (current + archived)
2. Extracts incomplete tasks
3. Assigns sequential IDs
4. Writes to appropriate status files (incomplete → ready, tasks from today → in-progress)
5. Builds INDEX.md
6. Adds completed tasks from finished files to INDEX.md as `done`

---

## Command Impact Summary

### Significantly rewritten

| Command | Key change |
|---------|-----------|
| `/start-day` | Focus selection from task store; no rolling forward |
| `/add-task` | Writes to `tasks/inbox.md` or `tasks/ready.md` with ID |
| `/dump` / `/brain-dump` | Same: task store first |
| `/review-day` | Marks done in task store, syncs changes |
| `/refine` | Supports subtask creation with IDs |

### New commands

| Command | Purpose |
|---------|---------|
| `/list-tasks` | Query task store by status, project, area, priority |
| `/update-task` | Change task status, timing, waiting info |
| `/new-from-template` | Instantiate a task/project template |
| `/migrate-tasks` | One-time v3→v4 migration |

### Minor updates

| Command | Change |
|---------|--------|
| `/link-task` | Also updates task store + INDEX.md |
| `/new-project` | Supports unified templates |
| `/new-area` | Supports sub-area creation |
| `/update-area` | Supports sub-area management |
| `/setup` | Creates `tasks/` directory |

---

## Philosophy

The persistent task store is a **UX experiment** for what could become a real application. By validating workflows and the data model in markdown-first with Claude, we get a battle-tested spec. If the system works well in markdown, migrating to a proper app means implementing a proven design rather than guessing at requirements.

Design principles remain unchanged:
- **Reduce friction** — task capture is still fast; IDs are auto-assigned
- **Create clarity** — lifecycle states make task status explicit
- **Bias toward action** — focus selection surfaces what matters today
- **Emotional safety** — data without judgment
- **Don't invent** — only track what the user explicitly mentions
