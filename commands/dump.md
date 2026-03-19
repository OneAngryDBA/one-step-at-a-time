---
description: "Quick task capture — paste or type tasks, get them organized and added to the task store"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Quick Task Capture

**Mode: QUICK CAPTURE**

A streamlined version of `/brain-dump` for when you just want to get tasks into the system fast.

## Step 1: Capture
Accept the user's input — could be a single task, a short list, or a quick thought.

No need to say "Ready — just start dumping." Just take what they give and organize it.

## Step 2: Organize
1. Extract tasks from input (don't invent or add anything not mentioned)
2. Format using OStaaT v4.0 task format:
   - Priority: 🔴 High / 🟡 Medium / 🟢 Low (default to 🟡 if unclear)
   - Timing: ⏰ Now / ⏭️ Next / 📅 Later (default to ⏭️ if unclear)
   - Time: 🔧 active minutes (required)
   - Energy label: if `energy.dimensions` configured, suggest appropriate label (optional)
3. Show the formatted tasks
4. Ask only if something is genuinely ambiguous — skip clarification for obvious items

## Step 3: Link to Projects or Areas (Optional)

If `projects.enabled` or `areas.enabled` in config:

1. Read active projects from `PROJECTS.md` (if exists)
2. Read areas from `AREAS.md` (if exists)
3. For each task (or in batches if many tasks), ask: "Link to a project or area?"
   - Show active projects with #tags
   - Show areas with @tags (including sub-areas)
   - Include "None" or "Skip" option
   - Allow batch selection: "Apply same project/area to all tasks?" if relevant
   - **Mutual exclusivity:** If user picks a project, no area tag needed (project's area provides context). A task gets either `#project-tag` or `@area-tag`, never both.

4. Add tags to selected tasks
5. Update project progress if `projects.autoCalculateProgress` enabled

If no active projects/areas or user skips, proceed to next step.

## Step 4: Assign IDs & Write to Task Store

1. Read `ostaat.json` to get `tasks.nextId`
2. For each task:
   a. Assign ID: `t-{nextId}` (zero-padded to 3 digits)
   b. Increment `nextId`
   c. Add `Created: YYYY-MM-DD` (today's date)
   d. Write to `tasks/ready.md` (or `tasks/inbox.md` if details are sparse)
   e. Add row to `tasks/INDEX.md`

3. Save updated `tasks.nextId` to `ostaat.json`

## Step 5: Optionally Add to Today's Focus

If today's daily file (`YYYY-MM-DD-todo.md`) exists:
1. Ask: "Add any of these to today's focus?"
   - **All** → copy all tasks to daily file, move to `tasks/in-progress.md`, add today to `Focus-dates`
   - **Pick** → let user choose which ones
   - **None** → tasks stay in their status files for next `/start-day`

If today's daily file doesn't exist:
- Inform user: "Tasks saved to the task store. Run `/start-day` to select today's focus."

## Step 6: Show Result

Show the final list with assigned IDs:
```
✅ 3 tasks added to task store:

- [t-042] 🟡 Medium ⏭️ Next 🔧 30 min @health — Schedule dentist
- [t-043] 🔴 High ⏰ Now 🔧 45 min #api-redesign — Fix auth endpoint
- [t-044] 🟢 Low 📅 Later 🔧 15 min — Update LinkedIn
```

Brief confirmation — done.

**Philosophy:** Speed over ceremony. Get tasks captured and move on. Use `/brain-dump` for the full guided flow with detailed clarification.
