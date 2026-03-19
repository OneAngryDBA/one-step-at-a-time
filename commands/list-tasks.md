---
description: "Query the task store by status, project, area, priority, or energy type"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

# Task: List Tasks from Task Store

**Mode: READ-ONLY** (no lock needed)

## Usage

Can be invoked with filters or interactively:
- `/list-tasks` — show all non-done tasks grouped by status
- `/list-tasks --status ready` — show only ready tasks
- `/list-tasks --status in-progress,waiting` — show multiple statuses
- `/list-tasks --tag @health` — filter by area tag
- `/list-tasks --tag #api-redesign` — filter by project tag
- `/list-tasks --priority high` — filter by priority
- `/list-tasks --energy deep` — filter by energy type
- `/list-tasks --all` — include done tasks from INDEX.md
- `/list-tasks --inbox` — shorthand for --status inbox

Filters can be combined: `/list-tasks --status ready --tag @work --priority high`

## Step 1: Parse Filters

Read any provided arguments and determine:
- **Status filter**: which status files to read (default: all non-done)
- **Tag filter**: #project or @area tag to match
- **Priority filter**: 🔴/🟡/🟢
- **Energy filter**: match energy label
- **Include done**: whether to show done tasks from INDEX.md

## Step 2: Read Task Store

**For status-filtered queries:**
1. Read the relevant `tasks/{status}.md` file(s)
2. Parse tasks and apply tag/priority/energy filters
3. Count results

**For --all (including done):**
1. Read `tasks/INDEX.md` for done task summaries
2. Read active status files for full details of non-done tasks

**For tag-only queries:**
1. Read `tasks/INDEX.md` first to find matching task IDs
2. Read only the relevant status files for full details

## Step 3: Display Results

### Default view (grouped by status)

```
📋 Task Store — 12 open tasks

In Progress (3):
- [t-017] 🔴 High ⏰ Now 🔧 45 min #api-redesign
  API endpoint refactoring
  Focus-dates: 2026-03-17, 2026-03-18, 2026-03-19

- [t-022] 🟡 Medium ⏭️ Next 🔧 30 min @health
  Schedule dentist appointment

- [t-030] 🔴 High ⏰ Now 🔧 120 min #monthly-close
  Monthly financial close (1/4 done)

Ready (5):
- [t-035] 🟡 Medium ⏭️ Next 🔧 60 min ⚡ Admin
  Update project documentation

- [t-036] 🟢 Low 📅 Later 🔧 15 min
  Clean up old test files
  ...

Waiting (2):
- [t-019] 🟡 Medium 🔧 30 min #tax-filing
  Review accountant response
  Waiting-on: Accountant email
  Expected: 2026-03-22

Inbox (1):
- [t-025] 🔧 15 min
  Look into monitoring tool

Paused (1):
- [t-028] 🟢 Low 🔧 45 min
  Research new testing framework
  Paused-reason: Waiting for Q2 planning
```

### Filtered view

When filters are applied, show a flat list with the filter noted:

```
📋 Tasks tagged @health — 3 tasks

- [t-022] 🟡 Medium ⏭️ Next 🔧 30 min (ready)
  Schedule dentist appointment

- [t-038] 🔴 High ⏰ Now 🔧 60 min (in-progress)
  Annual physical prep

- [t-001] (done)
  Set up exercise routine
```

### Summary stats

At the bottom, show:
```
Total: 12 open | 5 in-progress | 3 ready | 2 waiting | 1 inbox | 1 paused
Time: 🔧 445 min (~7.4 hrs) estimated across open tasks
```

## Step 4: Offer Actions

After displaying, suggest:
- "Use `/update-task t-NNN <status>` to change a task's status"
- "Use `/start-day` to select today's focus"
- "Use `/add-task` or `/dump` to add new tasks"

**Philosophy:** Fast, filterable visibility into all open work. Read-only — no changes made.
