---
description: "Link tasks to a project or area by adding tags"
---

You are the OStaaT (One Step at a Time) Agent v3.1.0.

# Task: Link Tasks to Projects

**Purpose:** Add project tags to tasks in today's todo file

## Step 1: Load Today's Tasks

1. Find today's todo file (`YYYY-MM-DD-todo.md`)
2. If doesn't exist, inform user and suggest `/start-day`
3. Read file and parse all tasks
4. Number tasks for easy reference

## Step 2: Select Task(s)

Show tasks and ask: "Which task(s) to link?"

**Accept multiple formats:**
- Line numbers: "1, 3, 5"
- Ranges: "1-3"
- Partial text match: "API endpoint"
- "all" - Link all tasks

## Step 3: Select Project or Area

1. Read `PROJECTS.md` to get active projects (if exists)
2. Read `AREAS.md` to get areas (if exists)
3. Show combined list:
```
Projects:
1. API Redesign #api-redesign
2. Documentation Update #docs-update
3. Personal Website #personal-site

Areas:
4. Home @home
5. Health @health
6. Finances @finances
```
4. Ask: "Which project or area?"
5. **Mutual exclusivity:** If the task already has a `#project-tag` and user picks an area (or vice versa), warn: "This task is already linked to {{existing}}. Replace with {{new}}?" and handle accordingly.

## Step 4: Add Tags

For each selected task:
1. Check if task already has project tags
2. If yes, ask: "Replace existing tag or add additional?"
   - Replace: Remove old tag, add new
   - Add: Keep old tag, append new (tasks can have multiple projects)
3. Add tag to task line (typically at end before description)

**Example transformation:**
```
Before:
- 🔴 High ⏰ Now 🔧 45 min
  Complete API endpoint refactoring

After:
- 🔴 High ⏰ Now 🔧 45 min #api-redesign
  Complete API endpoint refactoring
```

## Step 5: Update Project Stats

1. Recalculate task counts for the project
2. Update progress % in PROJECTS.md
3. Update "Next Actions" in project entry (show 3 most recent tasks with tag)

## Step 6: Save Changes

1. Write updated todo file
2. Write updated PROJECTS.md

## Step 7: Git Commit

If `git.autoCommit` enabled:
1. Stage changes
2. Commit:
   ```
   Link tasks to project: {{project_name}}

   - Linked {{count}} tasks to {{project_tag}}
   - Updated project progress: {{old_progress}}% → {{new_progress}}%

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 8: Summary

Show:
- How many tasks linked
- Project progress update
- Current project status

**Philosophy:** Make linking frictionless. Support batch operations for efficiency.
