---
description: "Link tasks to a project or area by adding tags — updates task store and daily file"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Link Tasks to Projects or Areas

**Purpose:** Add project tags or area tags to tasks — updating both the task store and daily file.

## Step 1: Select Task(s)

**Option A: Task ID provided** (e.g., `/link-task t-017`)
1. Look up task in `tasks/INDEX.md`
2. Read from its status file

**Option B: No ID provided**
1. If today's daily file exists, show tasks from it
2. Otherwise, show tasks from the task store (in-progress + ready)
3. Ask: "Which task(s) to link?"

**Accept multiple formats:**
- Task IDs: "t-017, t-022, t-035"
- Line numbers from daily file: "1, 3, 5"
- Ranges: "t-017 to t-022"
- Partial text match: "API endpoint"
- "all" — Link all listed tasks

## Step 2: Select Project or Area

1. Read `PROJECTS.md` to get active projects (if exists)
2. Read `AREAS.md` to get areas (if exists)
3. Show combined list:
```
Projects:
1. API Redesign #api-redesign
2. Documentation Update #docs-update
3. Personal Website #personal-site

Areas:
4. Work @work
   ├── Engineering @work/engineering
   └── Management @work/management
5. Home @home
   ├── Maintenance @home/maintenance
   └── Garden @home/garden
6. Health @health
```
4. Ask: "Which project or area?"
5. **Mutual exclusivity:** If the task already has a `#project-tag` and user picks an area (or vice versa), warn: "This task is already linked to {{existing}}. Replace with {{new}}?" and handle accordingly.

## Step 3: Update Task Store

For each selected task:
1. Read task from its status file
2. Add or replace tag on the task line
3. Write updated task back to the status file
4. Update `tasks/INDEX.md` tag column

**Example transformation in status file:**
```
Before:
- [t-017] 🔴 High ⏰ Now 🔧 45 min
  Complete API endpoint refactoring

After:
- [t-017] 🔴 High ⏰ Now 🔧 45 min #api-redesign
  Complete API endpoint refactoring
```

## Step 4: Update Daily File

If the task is in today's daily file (`YYYY-MM-DD-todo.md`):
1. Find the task by ID
2. Add/replace the tag
3. Save the daily file

## Step 5: Update Project Stats

If a `#project-tag` was applied:
1. Recalculate task counts for the project (scan INDEX.md for matching tags)
2. Update progress % in PROJECTS.md
3. Update "Next Actions" in project entry

## Step 6: Git Commit

If `git.autoCommit` enabled:
1. Stage changes
2. Commit:
   ```
   Link tasks to {{project_or_area}}: {{tag}}

   - Linked {{count}} task(s): {{task_ids}}
   {{if project: - Updated project progress: {{old_progress}}% → {{new_progress}}%}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 7: Summary

Show:
- How many tasks linked and to what
- Project progress update (if applicable)
- Current project/area status

**Philosophy:** Make linking frictionless. Support batch operations. Task store is always the source of truth — daily file is synced.
