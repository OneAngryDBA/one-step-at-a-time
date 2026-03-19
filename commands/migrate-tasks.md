---
description: "Extract tasks from daily files into the persistent task store — called by /upgrade or run standalone"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Migrate Tasks from Daily Files to Task Store

**Mode: CO-PILOT**

This command extracts tasks from existing daily files and writes them to the persistent task store. It can be run as part of `/upgrade` (v3→v4) or standalone at any time to import tasks from daily files.

## Step 0: Pre-flight Checks

1. Verify `ostaat.json` exists and has v4.0 config (tasks section present)
2. Check if `tasks/` directory exists with status files and INDEX.md
   - If not, create them (same as `/setup` Step 4c)
3. Check if INDEX.md already has entries
   - If yes, warn: "Task store already has entries. Continue? (This may create duplicates)"
   - Let user choose to proceed or abort

## Step 1: Scan Daily Files

Read all daily todo files, in order:
1. Today's file: `YYYY-MM-DD-todo.md` (current)
2. Recent files (not yet archived)
3. Archived files in `archive/` (if user opts in)

Ask: "Migrate from which files?"
- **Today only** — just today's todo file
- **Recent files** — today + any un-archived daily files
- **All files** — include archived files (comprehensive migration)

## Step 2: Extract Tasks

For each daily file:

### 2a: Parse incomplete tasks (from Tasks section)
Extract tasks that are NOT in the Completed section:
- Parse priority, timing, time estimates, due dates, tags
- Capture dependencies, context, carry-over notes
- Note which file(s) the task appeared in (for Focus-dates)

### 2b: Parse completed tasks (from finished files)
For each `YYYY-MM-DD-finished.md`:
- Extract completed tasks
- These will be added to INDEX.md as `done` status only (not to status files)

### 2c: Deduplicate
Tasks that were carried forward appear in multiple daily files. Deduplicate by:
1. Match on description text (normalized — lowercase, trimmed)
2. If same task appears in multiple files, keep the latest version (most metadata)
3. Collect all dates it appeared in as `Focus-dates`
4. Parse `📌 Carried over from:` notes to reconstruct the history

## Step 3: Show Migration Preview

```
📋 Migration Preview:

Incomplete tasks found: 12
- From today (2026-03-19): 5 tasks
- From 2026-03-18: 3 tasks (2 duplicates of today's)
- From 2026-03-17: 4 tasks (3 duplicates)
- Unique incomplete tasks: 8

Completed tasks found: 23 (across 5 finished files)

Proposed assignment:
- → in-progress: 5 (from today's file)
- → ready: 3 (incomplete, not in today's file)
- → INDEX.md as done: 23

Next ID will be: t-031 (after assigning t-001 through t-030 + subtasks)
```

Ask: "Proceed with migration? (You can review each task before confirming)"

## Step 4: Assign IDs & Write to Task Store

**If user wants to review:**
Show each unique incomplete task and confirm:
- Proposed status (in-progress if in today's file, ready otherwise)
- Ask: "Correct, or change? (ready / in-progress / waiting / paused / skip)"

**For each task:**
1. Assign sequential ID from `tasks.nextId`
2. Convert from v3 format to v4 format:
   - Add `[t-NNN]` ID prefix
   - Convert `📌 Carried over from:` notes to `Focus-dates:` list
   - Remove carry-over notes (no longer needed)
   - Add `Created:` date (earliest appearance date or today if unknown)
   - Keep all existing metadata (priority, timing, estimates, tags, dependencies, context)
3. Write to appropriate status file
4. Add to INDEX.md

**For completed tasks:**
- Add to INDEX.md with status `done` and summary
- Do NOT write to status files (they're already in finished files)

## Step 5: Update Config

1. Set `tasks.nextId` to the next available number
2. Save `ostaat.json`

## Step 6: Update Today's Daily File (if exists)

If today's todo file exists:
1. Rewrite it with task IDs added to each task
2. This makes the daily file consistent with the task store

## Step 7: Git Commit

If `git.autoCommit` enabled:
1. Stage all changes
2. Commit:
   ```
   Migrate v3→v4: extract tasks to persistent store

   - Incomplete tasks: {{N}} ({{in-progress}} in-progress, {{ready}} ready)
   - Completed tasks: {{N}} indexed
   - ID range: t-001 to t-{{max}}
   - Source files: {{count}} daily files scanned

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 8: Summary

```
✅ Migration complete!

  Tasks migrated: 8 incomplete + 23 completed indexed
  Task IDs: t-001 through t-031
  Status files: 5 in-progress, 3 ready
  INDEX.md: 31 entries

  Your task store is now the source of truth.
  Daily files will be generated from the store going forward.

  Next: Run /start-day to see your tasks in the new flow.
```

## Important Notes

- This migration is **additive** — it doesn't delete or modify existing daily files
- Carry-over notes are converted to Focus-dates (cleaner representation)
- If you run this again, it may create duplicates — check INDEX.md first
- After migration, `/start-day` will use the task store instead of rolling forward

**Philosophy:** Clean transition. Preserve all task history. Don't break anything — the old daily files remain as-is for reference.
