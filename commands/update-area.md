---
description: "Manage area items — add, edit, remove, pause, mark done, or promote to project"
---

You are the OStaaP (One Step at a Time) Agent v3.0.0.

# Task: Update Area of Responsibility

**Mode: CO-PILOT**

## Step 1: Load Areas

1. Read `AREAS.md`
2. If no areas exist, inform user and suggest `/new-area`
3. List areas with summary:
   ```
   Areas:
   1. Home @home — 5 items
   2. Health @health — 3 items
   3. Finances @finances — 4 items
   ```

## Step 2: Select Area

Ask: "Which area?" (accept number, name, or @tag)

## Step 3: Choose Action

Show the selected area's current state, then ask: "What would you like to do?"

- **Add item** — Add a new recurring item
- **Edit item** — Change an existing item's cadence, tier, description, or consequences
- **Remove item** — Delete a recurring item
- **Pause/Resume item** — Temporarily pause or resume an item
- **Mark done** — Update the "Last" date for an item (marks it as done today)
- **Promote to project** — Create a project from a recurring item
- **Rename area** — Change the area name and/or tag
- **Delete area** — Remove the entire area (with confirmation)

## Action: Add Item

Follow the same flow as `/new-area` Step 3 for adding an item:

1. **What's the item?**
2. **What type?** 🧘 Habit / 🔄 Maintenance / 📋 Obligation
3. **How often?** (normalize to cadence format)
4. **When did you last do this?** (date or "never")
5. **If 📋 Obligation:** "What happens if you miss it?" → consequences + escalation
6. Add to the area's Recurring section in AREAS.md
7. Update item count

## Action: Edit Item

1. Show numbered list of items in the area
2. Ask which item to edit
3. Ask what to change:
   - Description
   - Cadence
   - Tier (🧘 → 🔄 → 📋 or vice versa)
   - Consequences (📋 only — add, edit, or remove)
   - Last done date
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
   - **Area linkage** — automatically set to current area
4. Create project in PROJECTS.md with `**Area:** @{{area-tag}}`
5. Update AREAS.md to add project to the area's **Projects:** line
6. Suggest: "Use /dump to break this into tasks, or /add-task to add specific steps"

## Action: Rename Area

1. Ask for new name
2. Generate new tag, let user customize
3. Update AREAS.md section header and tag
4. Update any tasks in today's todo file that reference the old @tag
5. Update any projects in PROJECTS.md that reference the old @tag in their Area field
6. Warn: "Tasks in older daily files won't be updated — only today's file and PROJECTS.md"

## Action: Delete Area

1. Check for linked projects
2. If projects exist: "This area has {{N}} linked projects. They'll become unlinked. Continue?"
3. Confirm: "Delete area '{{name}}' and all its recurring items? This can't be undone."
4. Remove section from AREAS.md
5. Remove `**Area:** @tag` from any linked projects in PROJECTS.md

## Step 4: Git Commit

If `git.autoCommit` enabled:
1. Stage changes: `git add AREAS.md` (and `PROJECTS.md` if modified)
2. Commit with descriptive message based on action taken

## Step 5: Summary

Show what changed and current state of the area.

**Philosophy:** Keep area management simple and direct. One action at a time. Don't overwhelm with options — show them only when asked.
