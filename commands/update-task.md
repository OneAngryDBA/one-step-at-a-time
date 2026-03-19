---
description: "Change task status, timing, waiting info, or move between lifecycle states"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Update Task Status or Details

**Mode: CO-PILOT**

## Usage

Can be invoked with arguments or interactively:
- `/update-task t-017 ready` — move task t-017 to ready status
- `/update-task t-017 waiting` — move to waiting (will ask for Waiting-on info)
- `/update-task t-017` — interactive mode, show options
- `/update-task` — interactive mode, list tasks first

## Step 1: Identify Task

**If task ID provided as argument:**
1. Look up task ID in `tasks/INDEX.md`
2. If not found, inform user: "Task t-NNN not found in the index."
3. If found, read the task from its current status file

**If no task ID provided:**
1. Ask: "Which task? (provide t-NNN ID, or I can show your tasks)"
2. If user asks to see tasks, show a summary from INDEX.md:
   ```
   In Progress (3):
   - [t-017] 🔴 High — API endpoint refactoring
   - [t-022] 🟡 Medium — Schedule dentist
   - [t-030] 🔴 High — Monthly financial close (1/4 done)

   Ready (5):
   - [t-035] 🟡 Medium — Update documentation
   - [t-036] 🟢 Low — Clean up test files
   ...
   ```
3. User selects task

## Step 2: Choose Action

**If target status provided as argument:**
- Proceed directly to Step 3 with the specified status

**If no target status provided:**
Show current task details and ask: "What would you like to do?"

- **Change status** — Move to a different lifecycle state
  - `inbox` / `ready` / `in-progress` / `waiting` / `paused` / `done`
- **Change priority** — 🔴 High / 🟡 Medium / 🟢 Low
- **Change timing** — ⏰ Now / ⏭️ Next / 📅 Later
- **Update time estimate** — Adjust 🔧 active or 🕓 passive time
- **Update energy label** — Change or add energy type
- **Add/update due date** — Set or change 📅 deadline
- **Add/update lead time** — Set Lead: Nd or Start: YYYY-MM-DD
- **Update dependencies** — Add or remove dependency references
- **Add context** — Add notes, resources, links
- **Change tag** — Re-link to different project or area

## Step 3: Execute Status Transition

### Moving between status files

1. **Read** the task from its current status file
2. **Remove** the task from that file
3. **Apply** any status-specific metadata changes (see below)
4. **Write** the task to the new status file (in correct sort order)
5. **Update** `tasks/INDEX.md` with new status
6. If today's daily file exists and task was in it, update accordingly

### Status-specific metadata

**→ waiting:**
- Ask: "What is this task waiting on?" → set `Waiting-on:`
- Set `Waiting-since: YYYY-MM-DD` (today)
- Ask: "When do you expect it to be resolved?" → set `Expected: YYYY-MM-DD` (optional)

**→ paused:**
- Ask: "Why are you pausing this?" → set `Paused-reason:`

**→ in-progress:**
- Add today to `Focus-dates` if not already present
- If today's daily file exists, add task to it

**→ done:**
- Remove from current status file
- Update INDEX.md to status `done`
- Write to today's `YYYY-MM-DD-finished.md`
- If task has `@area-tag` and is a recurring item, update AREAS.md Last date
- If task is a subtask, update parent's progress count `(N/M done)`
- If all sibling subtasks are done, auto-complete the parent

**→ ready (from in-progress):**
- Remove today from `Focus-dates` if task was just added today
- If in today's daily file, remove from it

### Executing detail changes (non-status)

For priority, timing, time estimate, energy, due date, lead time, dependencies, context, or tag changes:
1. Read the task from its current status file
2. Update the specified field(s)
3. Re-sort within the file if priority/timing/due date changed
4. If task is in today's daily file, update there too
5. If tag changed, update INDEX.md

## Step 4: Git Commit

If `git.autoCommit` enabled:
1. Stage changes to affected files (status files, INDEX.md, ostaat.json, daily files)
2. Commit with message:
   ```
   Update task: [t-NNN] → {{new_status}}

   - {{task summary}}
   - Previous status: {{old_status}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 5: Summary

Show the updated task:
```
✅ [t-017] moved: in-progress → waiting
   API endpoint refactoring
   Waiting-on: Database migration approval from Sarah
   Expected: 2026-03-22
```

**Philosophy:** Fast status transitions. The most common operation should be the fastest — changing a task's state should take seconds.
