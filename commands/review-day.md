---
description: "Review the day, mark tasks complete in the task store, sync changes, show upcoming items"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Review Day, Mark Complete, Sync Task Store

## Step 1: Load Today's Files & Task Store
1. Read today's todo file (`YYYY-MM-DD-todo.md`)
2. Read today's finished file (`YYYY-MM-DD-finished.md`)
3. Read `tasks/in-progress.md` for the authoritative task state
4. Show the user their current task list from the daily file

## Step 2: Review & Mark Complete

Ask: "Which tasks did you complete? (list task IDs like t-017, t-022 — or say 'mark them in the file')"

**Options:**
- User lists completed task IDs → mark them done
- User says "mark them in the file" → read the file again and look for tasks manually marked as complete
- User can also mark subtasks: "t-031 and t-032 are done" (subtasks of t-030)

## Step 3: Process Completions in Task Store

For each completed task:

### 3a: Full task completion
1. **Remove** from `tasks/in-progress.md`
2. **Update** `tasks/INDEX.md`: set status to `done`
3. **Add** to `YYYY-MM-DD-finished.md` with all original metadata
4. **Remove** from today's todo file (move to Completed section)

### 3b: Subtask completion
1. **Mark subtask as done** inline within the parent task (in the status file)
2. **Update parent progress count**: `(N/M done)`
3. If ALL subtasks are now done → auto-complete the parent (follow 3a for parent)
4. Update daily file to reflect subtask completion

### 3c: Area recurring item completion
If completed task has `@area-tag` and `🔄 Recurring` notation:
1. Read `AREAS.md`
2. Find the matching recurring item in the tagged area
3. Update its "Last" date to today
4. Recalculate its status
5. Save updated AREAS.md

### 3d: Project progress update
If completed task has `#project-tag` and `projects.autoCalculateProgress`:
1. Read `PROJECTS.md`
2. Recalculate progress for the affected project
3. Update task counts and progress percentage
4. Save updated PROJECTS.md

## Step 4: Handle Remaining In-Progress Tasks

For tasks still in `tasks/in-progress.md` that weren't completed:

Ask: "What about the remaining tasks?"
- **Keep in progress** → they stay in `in-progress.md`, tomorrow's `/start-day` will pre-select them
- **Move to waiting** → ask for Waiting-on info, move to `tasks/waiting.md`
- **Pause** → ask for reason, move to `tasks/paused.md`
- **Move to ready** → de-prioritize back to backlog

Note: **No rolling forward needed** — tasks persist in the store. Tomorrow's `/start-day` will show them automatically.

## Step 5: Check for Overdue Tasks (if `dueDates.checkOnReviewDay` enabled)

Scan remaining tasks in the store for:
1. Tasks with `📅 YYYY-MM-DD` where date < today (overdue)
2. Tasks with `📅 [today's date]` that weren't completed (due today, missed)

If found:
```
⚠️ Overdue/Due Today Tasks (2):

- [t-017] 🔴 High 🔧 45 min 📅 2026-03-19 (DUE TODAY) #api-redesign
  API endpoint refactoring — still in progress

- [t-035] 🟡 Medium 🔧 30 min 📅 2026-03-18 (1 day overdue)
  Update documentation — status: ready
```

Ask: "These tasks are overdue or due today. What would you like to do?"
- **Reschedule** → update due dates
- **Escalate** → change priority to 🔴 High, timing to ⏰ Now
- **Leave as-is** → will surface again at next `/start-day`

## Step 6: Show Upcoming Due Dates (if `dueDates.checkOnReviewDay` enabled)

Scan all active tasks in the store for upcoming due dates:
```
📅 Upcoming Due Dates:

Tomorrow (3):
- [t-040] 🔴 High 🔧 60 min 📅 2026-03-20 #api-redesign (in-progress)
- [t-041] 🟡 Medium 🔧 30 min 📅 2026-03-20 (ready)
- [t-042] 🟢 Low 🔧 15 min 📅 2026-03-20 (ready)

This Week (2):
- [t-050] 🔴 High 🔧 90 min 📅 2026-03-24 #docs-update (ready)
- [t-051] 🟡 Medium 🔧 45 min 📅 2026-03-23 (ready)
```

Ask: "Ready for tomorrow's deadlines?"
- **Yes, all set** → continue
- **Need to adjust** → which ones?

## Step 7: Recurring Items for Tomorrow (if `areas.enabled` and `areas.showInReviewDay`)

1. Read `AREAS.md`
2. Identify items due tomorrow or within the next 2 days
3. If any found:
   ```
   📅 Recurring items coming up:
   - @home: 🔄 Vacuum house (due tomorrow, Saturday)
   - @finances: 📋 Pay credit card (due in 2 days, hard deadline)
     ⚠️ If missed: Late fee ($29)
   ```
4. Ask: "Want to schedule any of these for a specific day?"
   - If yes → create task in `scheduled/YYYY-MM-DD-todo.md` for the chosen date
   - If no → informational only, `/start-day` will surface them when due

## Step 8: Summary

Report:
```
📊 Day Review:

  Completed: 5 tasks (🔧 ~3.5 hrs active time)
  Still in progress: 2 tasks
  Moved to waiting: 1 task
  Overdue handled: 2 tasks rescheduled
  Area items updated: @home (vacuum), @health (exercise 3/4)
  Upcoming: 3 due tomorrow, 2 this week

  Task store: 10 open | 2 in-progress | 5 ready | 2 waiting | 1 inbox
```

Ask: "Want to review remaining tasks or adjust priorities?"

**Philosophy:** Celebrate progress, maintain clarity. No judgment on incomplete tasks — they stay in the store and surface when appropriate. No more stale carry-over notes. The task store handles persistence.
