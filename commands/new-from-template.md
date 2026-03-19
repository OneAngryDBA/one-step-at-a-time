---
description: "Instantiate a unified task/project template — create tasks and optionally a project from a reusable template"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Instantiate Template

**Mode: CO-PILOT**

## Step 1: Select Template

**If template name provided as argument** (e.g., `/new-from-template monthly-close`):
1. Look for the template in `{templates.folder}/` (default: `templates/tasks/`)
2. If not found, check `templates/projects/` as fallback
3. If still not found, list available templates

**If no argument:**
1. List available templates from both `templates/tasks/` and `templates/projects/`:
   ```
   Task Templates:
   1. example-checklist — Monthly financial reconciliation

   Project Templates:
   2. default — Basic project structure
   3. client-project — Client work with budget tracking
   4. personal-goal — Personal goals with habits & rewards
   ```
2. Ask: "Which template?"

## Step 2: Read & Parse Template

Read the template file. Parse the frontmatter:

```markdown
# Template Name

**Description:** What this template does
**Creates project:** yes|no
**Variables:** var1, var2, var3
**Default tag:** #{{project-tag}} or @area-tag
**Area:** @area/sub (optional)
```

## Step 3: Collect Variables

For each variable listed in `**Variables:**`:
1. Ask the user for the value
2. Show the variable name and any context from the template

Example:
```
Template "Client Onboarding" needs:
- client-name: What's the client name?
- account-manager: Who is the account manager?
```

**Special variables:**
- `{{project-tag}}` — auto-generated from project name if `Creates project: yes`
- `{{today}}` — replaced with today's date (YYYY-MM-DD)
- `{{due-date}}` — ask the user for the deadline

## Step 4: Create Project (if applicable)

If `**Creates project:** yes`:

1. Read the `## Project` section of the template
2. Replace variables in all fields
3. Ask for any additional details not in the template:
   - Project name (may be derived from variables)
   - Due date (if not in template)
   - Priority (if not in template, use template default)
4. Generate project tag from name
5. Create project entry in PROJECTS.md (same format as `/new-project`)
6. Set `**Area:**` if specified in template

## Step 5: Create Tasks

Read the `## Tasks` section of the template. For each task:

1. Replace all `{{variable}}` placeholders with user-provided values
2. Assign task ID from `tasks.nextId` sequence
3. Add `Created: YYYY-MM-DD` (today)
4. Apply default tag from template (or project tag if project was created)
5. If task has subtasks in the template, assign IDs to those too

**Resolve dependencies:**
- Template dependencies reference other template tasks by description
- Map these to the newly assigned task IDs
- Example: `Dependencies: Pull bank statements` → `Dependencies: t-051`

**Write tasks:**
1. Write each task to `tasks/ready.md` (or `tasks/inbox.md` if sparse)
2. Update `tasks/INDEX.md` for each task
3. Update `tasks.nextId` in `ostaat.json`

## Step 6: Apply Due Date & Lead Time

If a due date was provided (or inherited from a recurring item trigger):

1. Set `Due: YYYY-MM-DD` on the parent task (or all tasks if no parent)
2. If tasks have `Lead:` fields in the template, calculate `Start` dates relative to the due date
3. Backward-plan: dependencies determine ordering, lead times determine when each must start

## Step 7: Offer Focus Selection

If today's daily file exists:
1. Ask: "Add any of these tasks to today's focus?"
   - **All** → move to in-progress, add to daily file
   - **Pick** → let user choose
   - **None** → tasks stay in ready for next `/start-day`

## Step 8: Git Commit

If `git.autoCommit` enabled:
1. Stage all changes (task store, INDEX.md, ostaat.json, PROJECTS.md if created)
2. Commit:
   ```
   Instantiate template: {{template-name}}

   - Created {{N}} tasks (t-NNN to t-NNN)
   {{if project: - Created project: {{project-name}} ({{#project-tag}})}}
   - Variables: {{var1}}={{value1}}, {{var2}}={{value2}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 9: Summary

```
✅ Template "Monthly Financial Close" instantiated!

  Tasks created: 4 (t-051 to t-054)
  Tag: @finances
  Status: ready
  Due: 2026-03-31

  - [t-051] 🔴 High 🔧 30 min — Pull bank statements for March 2026
  - [t-052] 🔴 High 🔧 30 min — Reconcile accounts for March
  - [t-053] 🟡 Medium 🔧 20 min — Generate March financial reports
  - [t-054] 🟡 Medium 🔧 15 min — Submit March reports to accounting

  Use /start-day to select these for today's focus.
```

**Philosophy:** Templates eliminate repetitive task creation for recurring workflows. Define once, instantiate with variables. The template is a recipe; the instantiated tasks are independent entries in the task store.
