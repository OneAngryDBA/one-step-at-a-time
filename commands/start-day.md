---
description: "Initialize today's daily files from the task store — select focus, triage inbox, check reminders"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Initialize Today's Daily Files from Task Store

## Step 1: Check for existing today's files

- Look for `YYYY-MM-DD-todo.md` and `YYYY-MM-DD-finished.md` for today's date
- If they already exist, inform the user and ask if they want to reinitialize
- If reinitializing, read the existing daily file to understand current state

## Step 2: Read Task Store

Read the following files from `{dataDir}/tasks/`:

### 2a: In-Progress tasks
Read `in-progress.md`. These are tasks the user was working on (possibly yesterday).
- **Pre-select all by default** (if `tasks.autoSelectInProgress` is true)
- Show each with its Focus-dates history

### 2b: Ready tasks
Read `ready.md`. These form the actionable backlog.
- Sort by: priority (🔴 → 🟡 → 🟢), then timing (⏰ → ⏭️ → 📅), then due date
- **Highlight** tasks where:
  - `Start ≤ today` (explicit start date reached)
  - `Due - Lead ≤ today` (lead time triggered)
  - Due date is today or past (overdue)
  - Due date is within `dueDates.bufferDays` (due soon)

### 2c: Waiting tasks
Read `waiting.md`. Flag any where `Expected` date has passed:
```
⏳ Waiting tasks with passed expected dates:
- [t-019] Review accountant response — Expected: 2026-03-15 (4 days ago)
  Waiting-on: Accountant email
```
Ask: "Follow up on any of these? (move to in-progress or back to ready)"

### 2d: Inbox tasks (if `tasks.showInboxAtStartDay` is true)
Read `inbox.md`. Show untriaged items for quick triage:
```
📥 Inbox (3 untriaged):
- [t-025] Look into monitoring tool
- [t-026] Research CI options
- [t-027] Check on license renewal
```
Ask: "Triage these now? (assign priority/timing to move to ready, or skip)"
- For each triaged task: assign priority, timing, time estimate → move to `ready.md`, update INDEX.md
- Skip option: leave in inbox for later

## Step 3: Pull scheduled tasks for today

- Check `scheduled/` folder for a file matching today's date (`YYYY-MM-DD-todo.md`)
- If found:
  1. Read the tasks from the scheduled file
  2. Assign task IDs from `tasks.nextId` sequence
  3. Write to `tasks/inbox.md` or `tasks/ready.md` (ready if they have full details)
  4. Update `tasks/INDEX.md`
  5. Increment `tasks.nextId` in `ostaat.json`
  6. Remove the scheduled file
  7. Show: "📅 Pulled N scheduled tasks into the task store"

## Step 4: Check for overdue tasks (if `dueDates.checkOnStartDay` enabled)

Scan all active status files for tasks with `📅 YYYY-MM-DD` where date < today:
```
⚠️ Overdue Tasks (3):

- [t-017] 🔴 High 🔧 45 min 📅 2026-02-28 (2 days overdue) #api-redesign
  Complete API endpoint refactoring

- [t-035] 🟡 Medium 🔧 30 min 📅 2026-03-01 (1 day overdue)
  Update documentation
```

Ask: "What would you like to do with overdue tasks?"
- **Reschedule all** → ask for new date, update due dates
- **Reschedule individually** → one at a time
- **Mark for today** → update timing to ⏰ Now
- **Review later** → skip for now

## Step 5: Show active projects (if `projects.showInStartDay` enabled)

Read `PROJECTS.md` if it exists. Display active projects:
```
Active Projects (3):
- 🔴 API Redesign (65%) — 12/24 tasks — Due: 2026-03-15
- 🟡 Documentation Update (30%) — 5/18 tasks — Due: 2026-04-01
- 🟢 Personal Website (10%) — 2/15 tasks — Due: 2026-05-01
```

Show task store stats for projects:
```
Tasks in store with project tags:
- #api-redesign: 4 ready, 2 in-progress
- #docs-update: 3 ready
```

## Step 6: Area reminders (if `areas.enabled` and `areas.showInStartDay`)

Read `AREAS.md` if it exists. Calculate current status for all recurring items.

```
🔔 Area Reminders:

@home:
- 🔴 📋 Change HVAC filter — overdue 5 days
  ⚠️ Reduced efficiency, potential system damage
- ⚠️ 🔄 Vacuum house — due today

@health:
- 🟡 🧘 Exercise — 2/4 this week

@finances:
- 🟡 🔄 Review subscriptions — last done 35 days ago (every month)
```

For overdue 📋 Obligations, always show the consequence line.

**Template suggestions:** If any recurring item has a `Template:` field and is now due (or within Lead time):
```
📋 Monthly financial close is due (Due: 2026-03-31, Lead: 3d)
   Template: monthly-close available — instantiate now?
```
If user confirms, run template instantiation (see `/new-from-template` flow).

Ask: "Add any of these to today's tasks?"
- **Pick which ones** → create tasks in task store with @area-tag and 🔄 Recurring notation, then add to today's focus selection
- **Add all due items** → batch create
- **Schedule for a future day** → ask for date, create in `scheduled/YYYY-MM-DD-todo.md`
- **Skip** → proceed

## Step 7: Focus Selection

Present the combined view of tasks available for today:

```
📋 Select today's focus:

Already in progress (auto-selected):
  ✅ [t-017] 🔴 High 🔧 45 min #api-redesign — API refactoring
  ✅ [t-030] 🔴 High 🔧 120 min #monthly-close — Financial close (1/4)

Ready — highlighted (lead time / due soon):
  ☐ [t-035] 🟡 Medium 🔧 60 min — Update docs (due in 2 days)
  ☐ [t-040] 🔴 High 🔧 30 min — Prepare quarterly review (start date: today)

Ready — backlog:
  ☐ [t-036] 🟢 Low 🔧 15 min — Clean up test files
  ☐ [t-038] 🟡 Medium 🔧 30 min @health — Annual physical prep
  ...

Area items added:
  ☐ [t-048] 🟡 Medium 🔧 20 min @home — Vacuum house
```

Ask: "Confirm today's focus? (select/deselect by ID, or 'confirm' to proceed)"

User can:
- Add items by ID: "add t-035, t-048"
- Remove items: "remove t-030"
- Confirm: "confirm" or "looks good"

## Step 8: Apply Focus Selection

For each selected task:
1. Move to `tasks/in-progress.md` (if not already there)
2. Add today's date to `Focus-dates`
3. Update `tasks/INDEX.md` status to `in-progress`

For any in-progress tasks NOT selected (de-selected):
1. Move back to `tasks/ready.md`
2. Update `tasks/INDEX.md` status to `ready`

## Step 9: Generate Daily Files

### Today's todo file (`YYYY-MM-DD-todo.md`):
```markdown
# Thursday, March 19, 2026

## Tasks

- [t-017] 🔴 High ⏰ Now 🔧 45 min #api-redesign
  API endpoint refactoring
  Focus-dates: 2026-03-17, 2026-03-18, 2026-03-19

- [t-030] 🔴 High ⏰ Now 🔧 120 min #monthly-close
  Monthly financial close (1/4 done)
  Due: 2026-03-31
  Focus-dates: 2026-03-19
  - [t-031] 🔧 30 min ⚡ Admin — Pull bank statements
  - [t-032] 🔧 30 min 🧠 Deep — Reconcile accounts
  - [t-033] 🔧 20 min ⚡ Admin — Generate reports
  - [t-034] 🔧 15 min ⚡ Admin — Submit to accounting

- [t-048] 🟡 Medium ⏭️ Next 🔧 20 min @home
  Vacuum house 🔄 Recurring

## Completed
```

### Today's finished file (`YYYY-MM-DD-finished.md`):
Empty with header and date, ready to receive completed tasks.

## Step 10: Check for old files to archive

List any todo/finished files that are 3+ days old. Suggest `/archive-old` if needed.

## Step 11: Load check (if `areas.loadCheck` enabled)

Count: today's total tasks, total 🔧 time, recurring items due/nudging, overdue obligations.
Compare against `loadThresholds`.

If any threshold exceeded:
```
📊 Load Check:
- 12 tasks today (threshold: 8)
- 6 recurring items due
- 3 obligations overdue

Options:
- Triage (defer some tasks back to ready)
- Move some tasks to tomorrow
- Proceed as-is
```

If no thresholds exceeded, don't display anything.

## Step 12: Summary

```
✅ Day initialized!

  Focus: 5 tasks (🔧 ~4.2 hrs)
  In progress: 5 | Ready: 8 | Waiting: 2 | Inbox: 1
  Area reminders: 3 items surfaced
  Files: 2026-03-19-todo.md, 2026-03-19-finished.md

  Use /dump to add tasks, /update-task to change status,
  /review-day when you're done.
```

**Philosophy:** The task store does the heavy lifting. No more rolling forward — tasks persist. Focus selection replaces copy-paste carry-over. You pick what matters today from your full backlog.
