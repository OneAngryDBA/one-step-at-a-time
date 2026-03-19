---
description: "Manage area items — add, edit, remove, pause, mark done, promote to project, manage sub-areas"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Update Area of Responsibility

**Mode: CO-PILOT**

## Step 1: Load Areas

1. Read `AREAS.md`
2. If no areas exist, inform user and suggest `/new-area`
3. List areas with summary (including sub-areas):
   ```
   Areas:
   1. Work @work — 8 items, 2 sub-areas
      ├── Engineering @work/engineering — 3 items
      └── Management @work/management — 2 items
   2. Home @home — 5 items, 2 sub-areas
      ├── Maintenance @home/maintenance — 3 items
      └── Garden @home/garden — 2 items
   3. Health @health — 3 items
   ```

## Step 2: Select Area

Ask: "Which area?" (accept number, name, or @tag — including sub-area tags like @work/engineering)

## Step 3: Choose Action

Show the selected area's current state, then ask: "What would you like to do?"

- **Add item** — Add a new recurring item
- **Edit item** — Change an existing item's cadence, tier, description, or consequences
- **Remove item** — Delete a recurring item
- **Pause/Resume item** — Temporarily pause or resume an item
- **Mark done** — Update the "Last" date for an item (marks it as done today)
- **Promote to project** — Create a project from a recurring item
- **Add sub-area** — Create a sub-area under this area (top-level areas only)
- **Move item** — Move a recurring item between area and sub-area
- **Rename area** — Change the area name and/or tag
- **Delete area** — Remove the entire area (with confirmation)

## Action: Add Item

Follow the same flow as `/new-area` Step 3 for adding an item:

1. **What's the item?**
2. **What type?** 🧘 Habit / 🔄 Maintenance / 📋 Obligation
3. **How often?** (normalize to cadence format)
4. **When did you last do this?** (date or "never")
5. **If 📋 Obligation:** "What happens if you miss it?" → consequences + escalation
6. **Template?** If relevant, link to a template in `templates/tasks/` with optional Lead time
7. Add to the area's Recurring section in AREAS.md
8. Update item count

## Action: Edit Item

1. Show numbered list of items in the area (and sub-areas)
2. Ask which item to edit
3. Ask what to change:
   - Description
   - Cadence
   - Tier (🧘 → 🔄 → 📋 or vice versa)
   - Consequences (📋 only — add, edit, or remove)
   - Last done date
   - Template reference
   - Lead time
4. Update the item in AREAS.md

## Action: Remove Item

1. Show numbered list of items
2. Ask which to remove
3. Confirm: "Remove '{{item}}'? This can't be undone."
4. Remove from AREAS.md
5. Update item count

## Action: Pause/Resume Item

1. Show items, ask which one
2. If currently active → set status to `⏸️ Paused`
3. If currently paused → recalculate status based on cadence and last done
4. Update in AREAS.md

## Action: Mark Done

1. Show items, ask which one (or accept as argument)
2. Update the "Last" date to today's date (`YYYY-MM-DD`)
3. Recalculate status (should become ✅ On track or ✅ Next: date)
4. For `Nx/week` items, increment this week's count
5. Update in AREAS.md

## Action: Promote to Project

1. Show items, ask which to promote
2. Explain: "This will create a project for the current instance. The recurring item stays in the area for next time."
3. Gather project details (similar to `/new-project` but pre-filled):
   - **Project name** — suggest based on item (e.g., "File 2026 Taxes")
   - **Description** — from the recurring item
   - **Due date** — from the item's next due date
   - **Area linkage** — automatically set to current area (including sub-area path)
4. Create project in PROJECTS.md with `**Area:** @{{area-tag}}`
5. Update AREAS.md to add project to the area's **Projects:** line
6. If the item has a `Template:` reference, suggest: "Instantiate the template to create tasks? (`/new-from-template`)"
7. Suggest: "Use /dump to break this into tasks, or /add-task to add specific steps"

## Action: Add Sub-Area

Only available for top-level areas:
1. Ask: "What sub-area name?"
2. Generate tag: `@parent/child`
3. Follow `/new-area` Step 3 for adding recurring items to the sub-area
4. Insert sub-area section under the parent in AREAS.md
5. Update parent's `**Sub-areas:** N` count

## Action: Move Item

Move a recurring item between a parent area and sub-area (or between sub-areas):
1. Show all items across the area and its sub-areas
2. Ask which item to move and where
3. Remove from source, add to destination
4. Update item counts for both

## Action: Rename Area

1. Ask for new name
2. Generate new tag, let user customize
3. Update AREAS.md section header and tag
4. If renaming a parent area, update all sub-area tags too (e.g., @work/engineering → @career/engineering)
5. Update any tasks in the task store (`tasks/*.md`) that reference the old @tag
6. Update `tasks/INDEX.md` for affected tasks
7. Update any projects in PROJECTS.md that reference the old @tag in their Area field
8. If today's daily file exists, update tags there too
9. Warn: "Tasks in archived daily files won't be updated — only the task store, today's file, and PROJECTS.md"

## Action: Delete Area

1. Check for linked projects and sub-areas
2. If sub-areas exist: "This area has {{N}} sub-areas. They'll also be deleted. Continue?"
3. If projects exist: "This area has {{N}} linked projects. They'll become unlinked. Continue?"
4. Confirm: "Delete area '{{name}}' and all its recurring items? This can't be undone."
5. Remove section from AREAS.md
6. Remove `**Area:** @tag` from any linked projects in PROJECTS.md
7. Note: Tasks in the task store that reference this area's tag are NOT deleted — they keep their tag as a historical reference

## Step 4: Git Commit

If `git.autoCommit` enabled:
1. Stage changes: `git add AREAS.md` (and `PROJECTS.md`, task store files if modified)
2. Commit with descriptive message based on action taken

## Step 5: Summary

Show what changed and current state of the area.

**Philosophy:** Keep area management simple and direct. One action at a time. Don't overwhelm with options — show them only when asked. Sub-areas add structure when the user needs it.
