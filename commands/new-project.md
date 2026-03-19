---
description: "Create a new project from scratch, template, or duplicate — with optional task creation from unified templates"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Create New Project

**Mode: CO-PILOT**

## Step 1: Source Selection

Ask: "How do you want to create this project?"
- **From scratch** — Full guided questions
- **From template** — Use existing template (project-only or unified task+project)
- **Duplicate existing** — Copy from active or archived project

## Step 2: Execute Based on Choice

### If "From scratch":
Proceed to Step 3 (Guided Questions)

### If "From template":
1. List available templates from both `templates/projects/` and `templates/tasks/`:
   - **Project-only templates** (`templates/projects/`):
     - default.md — Basic project structure
     - client-project.md — Client work with budget tracking
     - personal-goal.md — Personal goals with habits & rewards
   - **Unified templates** (`templates/tasks/`) — marked with `Creates project: yes`:
     - client-onboarding.md — Client onboarding with project + tasks
     - [Any user-created templates with `Creates project: yes`]

2. User selects template

3. **If unified template selected** (has `Creates project: yes`):
   - Redirect to `/new-from-template` flow which handles both project and task creation
   - That command will create the project in PROJECTS.md AND tasks in the task store

4. **If project-only template selected:**
   - Read template file
   - Proceed to Step 3, using template as base (pre-fill some fields)

### If "Duplicate existing":
1. List all projects (active from PROJECTS.md + archived from projects-archive/)
2. User selects project to duplicate
3. Read selected project
4. Ask: "Keep same details or modify?"
5. Proceed to Step 3, pre-filling with duplicated project data

## Step 3: Guided Questions

Ask one question at a time:

1. **Project name?**
2. **Brief description?**
3. **Main goals/objectives?** (can add multiple, ask "Add another goal?" after each)
4. **Priority?** (🔴 High / 🟡 Medium / 🟢 Low)
5. **Start date?** (default: today)
6. **Target end date?**
7. **Milestones?** (optional, format: "YYYY-MM-DD — Milestone name", can add multiple)
8. **Stakeholders?** (optional, can add multiple)
9. **Tags/categories?** (optional, e.g., "client-work", "personal", client name)
10. **Resources?** (optional, links/docs/contacts)
11. **Area?** (optional) If `areas.enabled` and `AREAS.md` exists:
    - "Does this project belong to an area?"
    - Show available areas with @tags (including sub-areas)
    - "None" option
    - If selected, will add `**Area:** @tag` to the project entry

**Template-specific questions:**
- If client-project template: Ask about client name, budget, hourly rate
- If personal-goal template: Ask about "why this matters", habits, rewards

## Step 4: Generate Project Tag

1. Convert project name to tag format:
   - Lowercase
   - Replace spaces with hyphens
   - Remove special characters
   - Examples: "API Redesign" → `#api-redesign`, "Client Onboarding" → `#client-onboarding`

2. Show generated tag
3. Ask: "Use this tag or specify custom tag?"

## Step 5: Create Project Files

1. **Add to PROJECTS.md:**
   - If PROJECTS.md doesn't exist, create it with headers
   - Add project under appropriate priority section
   - Use format:
   ```markdown
   ### {{Project Name}} {{#project-tag}}
   **Status:** 🟢 Active | **Progress:** 0% | **Due:** {{target_end_date}} {{if area: | **Area:** @area-tag}}

   **Goals:**
   {{goals list}}

   **Next Actions:** (none yet)

   ---
   ```

2. **If needed, create detailed project file** (optional, or only for archived):
   - Save in projects-archive/ for reference
   - Use selected template format
   - Fill in all placeholders

## Step 6: Offer Task Creation

After the project is created, ask:
"Want to add initial tasks for this project?"

- **Yes, from a task template** → Show templates from `templates/tasks/` that don't have `Creates project: yes` (task-only templates). If selected, instantiate with the new project's `#tag`
- **Yes, quick add** → Run a quick `/dump`-style capture with the project tag pre-applied
- **Yes, break it down** → Run a `/refine`-style breakdown to create subtasks
- **No, I'll add tasks later** → Done

## Step 7: Git Commit

If `git.autoCommit` enabled:
1. Stage changes: `git add PROJECTS.md` (and task store files if tasks were created)
2. Commit with message:
   ```
   New project: {{project_name}}

   - Priority: {{priority}}
   - Due: {{target_end_date}}
   - Tag: {{project_tag}}
   {{if area: - Area: @{{area-tag}}}}
   {{if tasks: - Initial tasks: {{count}} (t-NNN to t-NNN)}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 8: Summary

Show the created project with:
- Project name and tag
- Priority and timeline
- Goals
- Area linkage (if any)
- Initial tasks (if any, with IDs)
- Next steps: "Add tasks with {{#project-tag}} to link them. Use /dump, /add-task, or /new-from-template."

**Philosophy:** Make project creation easy but comprehensive. Don't overwhelm with questions — use sensible defaults. Unified templates bridge the gap between project creation and task planning.
