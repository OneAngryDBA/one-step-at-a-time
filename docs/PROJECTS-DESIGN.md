# Projects System Design

**Version:** 1.0.0
**Status:** Design Complete - Ready for Implementation

---

## Overview

Projects provide a higher-level organizational layer above daily tasks. Each project tracks goals, timelines, progress, and related tasks through tag-based linking.

---

## Project Structure

### Core Fields (Required)
- **Name** - Project identifier (also used for file naming)
- **Description** - What the project is about
- **Goals/Objectives** - What success looks like, measurable outcomes
- **Timeline**
  - Start date
  - Target end date
  - Milestones (key checkpoints with dates)

### Context Fields
- **Priority** - 🔴 High / 🟡 Medium / 🟢 Low
- **Status** - 🟢 Active / ⏸️ On Hold / ✅ Completed / 📦 Archived
- **Stakeholders** - People involved or interested
- **Tags/Categories** - Work, Personal, Client name, etc.

### Tracking Metrics (Auto-calculated)
- **Progress %** - 0-100% completion
- **Task Counts**
  - Total tasks with this project tag
  - Completed tasks
  - Remaining tasks
- **Time Tracking** - Sum of time estimates from tagged tasks

### Additional Sections
- **Resources** - Links, documents, repos, contacts
- **Notes/Journal** - Ongoing project notes and decisions

---

## File Structure

```
your-project/
├── PROJECTS.md                    # Active projects overview
├── projects-archive/              # Completed/archived projects
│   ├── project-name.md            # Individual archived project
│   ├── project-name-2.md          # Duplicate names get numbered
│   └── ...
├── templates/
│   ├── projects/                  # Project templates
│   │   ├── default.md
│   │   ├── client-project.md
│   │   └── personal-goal.md
│   └── tasks/                     # Task templates (future)
│       └── default.md
├── YYYY-MM-DD-todo.md            # Daily todos with project tags
└── ...
```

### PROJECTS.md Structure

```markdown
# Active Projects

## 🔴 High Priority

### Project Name #project-tag
**Status:** 🟢 Active | **Progress:** 65% | **Due:** 2026-03-15

**Goals:**
- Primary objective
- Secondary objective

**Next Actions:** (3 most recent incomplete tasks with this tag)
- Task 1
- Task 2
- Task 3

---

## 🟡 Medium Priority

...

## 🟢 Low Priority

...

## ⏸️ On Hold

...
```

### Individual Project File Structure (Archive)

```markdown
# Project Name

**Tags:** #project-tag
**Priority:** 🔴 High
**Status:** ✅ Completed
**Dates:** 2026-01-15 → 2026-03-15

---

## Description

[Project description]

## Goals/Objectives

- [ ] Goal 1
- [x] Goal 2
- [x] Goal 3

## Timeline & Milestones

- **Start Date:** 2026-01-15
- **Target End:** 2026-03-15
- **Actual End:** 2026-03-10

**Milestones:**
- [x] 2026-02-01 - Milestone 1
- [x] 2026-02-15 - Milestone 2
- [x] 2026-03-10 - Milestone 3

## Stakeholders

- Person 1 (role)
- Person 2 (role)

## Progress Tracking

**Final Stats:**
- Total tasks: 24
- Completed: 24
- Progress: 100%
- Total time: 🔧 840 min (14 hours)

## Resources

- [Link 1](url)
- [Document 2](url)
- Contact: person@example.com

## Notes

[Project notes and journal entries]

---

## Completed Tasks

[List of all completed tasks with this project tag]
```

---

## Task-Project Linking

### Tag Format

Tasks use hashtags to link to projects:
- `#project-tag` - Links to a specific project
- Multiple tags allowed if task relates to multiple projects

### Example in Daily Todo

```markdown
## Tasks

- 🔴 High ⏰ Now 🔧 45 min #api-redesign
  Complete API endpoint refactoring
  Dependencies: None
  Context: See #api-redesign project docs
```

### Tag Naming Convention

- Lowercase, hyphen-separated
- Keep concise (2-3 words max)
- Examples: `#api-redesign`, `#client-onboarding`, `#personal-fitness`

---

## Slash Commands

### /new-project

**Purpose:** Create a new project with guided questions

**Workflow:**
1. Ask: "Starting from scratch, template, or duplicate existing project?"
   - **Scratch** → Full guided questions
   - **Template** → List available templates, then customize
   - **Duplicate** → List existing/archived projects, then customize

2. Guided questions (if scratch or after template selection):
   - Project name?
   - Description?
   - Goals/objectives? (one at a time, add more?)
   - Priority? (🔴/🟡/🟢)
   - Start date? (default: today)
   - Target end date?
   - Milestones? (optional)
   - Stakeholders? (optional)
   - Tags/categories? (optional)
   - Resources? (optional)

3. Generate project tag (from name)
4. Add to PROJECTS.md
5. Show formatted project

### /list-projects

**Purpose:** Show all projects with status and progress

**Options:**
- Default: Show active projects only
- `--all` or `-a`: Include on-hold projects
- `--archived`: Show archived projects
- `--filter priority:high`: Filter by priority
- `--filter tag:client`: Filter by tag

**Output:**
```
Active Projects (3)

🔴 API Redesign #api-redesign
   Progress: 65% | 12/24 tasks | Due: 2026-03-15

🟡 Documentation Update #docs-update
   Progress: 30% | 5/18 tasks | Due: 2026-04-01

🟢 Personal Website #personal-site
   Progress: 10% | 2/15 tasks | Due: 2026-05-01
```

### /update-project

**Purpose:** Update project details, progress, or status

**Workflow:**
1. Show list of active projects (or specify project name/tag)
2. Ask: "What do you want to update?"
   - Status (Active/On Hold/Completed)
   - Priority
   - Timeline (dates, milestones)
   - Goals/objectives
   - Add/update notes
   - Recalculate progress (manual override)

3. Make updates
4. If status changed to Completed:
   - Move to projects-archive/
   - Create individual project file
   - Remove from PROJECTS.md

### /link-task

**Purpose:** Helper to add project tag to tasks in daily file

**Workflow:**
1. Ask: "Which task(s) to link?" (can reference by line number or partial text)
2. Show available project tags
3. Add selected tag to task(s)
4. Update project progress automatically

### /reopen-project

**Purpose:** Pull an archived project back to active

**Workflow:**
1. List archived projects
2. User selects one
3. Ask: "Keep same dates or update?"
4. Move back to PROJECTS.md as Active
5. Optionally keep archived copy with "(v1)" suffix

### /review-projects (Weekly)

**Purpose:** Weekly review of all project progress

**Workflow:**
1. Show all active projects with:
   - Progress since last week
   - Upcoming milestones
   - Blocked/stalled projects (no progress in 7+ days)

2. For each project, ask:
   - Update status?
   - Add notes?
   - Adjust timeline?

3. Generate weekly summary report

---

## Integration with Daily Todos

### /start-day Integration

**Added behavior:**
1. After rolling forward tasks
2. Show active projects section:
```
Active Projects (3):
- 🔴 API Redesign (65%) - 12/24 tasks
- 🟡 Documentation Update (30%) - 5/18 tasks
- 🟢 Personal Website (10%) - 2/15 tasks

Tasks with project tags being rolled forward:
- 2 tasks for #api-redesign
- 1 task for #docs-update
```

### /dump Integration

**Added behavior:**
1. After extracting candidate tasks
2. For each task, ask: "Link to a project? (skip if none)"
   - Show active projects + "None" option
   - Add tag if selected
3. Update project progress after adding tasks

### Auto-Calculate Progress

**Trigger:** After any command that adds/completes tasks
1. Count all tasks with project tag across:
   - Today's todo file
   - Yesterday's todo file (for context)
   - Finished files (last 7 days)
2. Calculate: `(completed / total) * 100`
3. Update PROJECTS.md

### Weekly Project Review

**New command:** `/review-week` or `/weekly-review`
- Review completed tasks by project
- Show project progress
- Identify stalled projects
- Suggest actions

---

## Duplicate Name Handling

When archiving projects with duplicate names:
1. Check if `project-name.md` exists in archive
2. If exists, append number: `project-name-2.md`
3. Keep incrementing: `project-name-3.md`, etc.
4. Log in project file: "This is iteration 2 of this project"

---

## Templates System

### Template Folder Structure

```
templates/
├── projects/
│   ├── default.md           # Basic project template
│   ├── client-project.md    # Client work template
│   ├── personal-goal.md     # Personal project template
│   └── ...                  # User can add custom templates
└── tasks/
    └── default.md           # Future: task templates
```

### Creating Custom Templates

Users can create new templates by:
1. Copying an existing template
2. Customizing fields, sections, and defaults
3. Saving in `templates/projects/`
4. Template becomes available in `/new-project`

---

## Configuration

Add to `todo-config.json`:

```json
"projects": {
  "enabled": true,
  "autoCalculateProgress": true,
  "showInStartDay": true,
  "archiveFolder": "projects-archive",
  "templatesFolder": "templates/projects",
  "weeklyReviewDay": "friday"
}
```

---

## Git Integration

Projects trigger commits:
- `/new-project` → "New project: [name]"
- `/update-project` → "Update project: [name] - [changes]"
- Project archived → "Archive project: [name]"
- `/reopen-project` → "Reopen project: [name]"

---

## Future Enhancements

- Project dependencies (one project blocks another)
- Gantt chart generation from milestones
- Budget/cost tracking per project
- Integration with Jira/GitHub projects (auto-import)
- Time-based project alerts (approaching deadline)

---

## Next Steps for Implementation

1. Create template files
2. Create slash commands
3. Update config file
4. Update existing commands (/start-day, /dump) for integration
5. Test end-to-end workflow
6. Document in README
