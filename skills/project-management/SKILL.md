---
name: project-management
description: This skill should be used when the user mentions projects, long-term goals, multi-task efforts, client work, launches, milestones, deadlines, or wants to track progress on larger initiatives. It also applies when the user describes something that sounds like a project (has multiple steps and an end goal) but has not created one yet, or asks about project status, progress, or planning. It should not be used for simple one-off tasks, daily routine questions, or chores/maintenance items (those belong to area-management).
---

# Project Management Skill

Help users manage projects effectively using OStaaT's project system with task store integration.

## Recognizing Project-Scale Work

A project is appropriate when:
- There are **multiple tasks** toward a **single goal**
- There's a **target end date** or deadline
- It's **not recurring** (recurring → area, not project)
- It needs **progress tracking**
- Examples: "Redesign the website", "File 2026 taxes", "Plan vacation", "Launch new product"

A project is NOT appropriate when:
- It's a single task (just add it to the task store)
- It's an ongoing responsibility (that's an area: @home, @health)
- It's a recurring chore (that's a maintenance item in an area)

## How to Help

### When user describes something that sounds like a project:
1. Recognize it: "That sounds like it could be a project — multiple steps with a clear goal"
2. Suggest: "Want to create a project for it? `/new-project` will walk you through it"
3. Mention: "If you have a template for this kind of work, `/new-from-template` can create the project with tasks"
4. Don't force it — if they just want tasks, help with tasks

### When user asks about project status:
1. Read `PROJECTS.md`
2. Read `tasks/INDEX.md` to get task counts by project tag
3. Show relevant project details (progress, task breakdown by status, deadlines)
4. Suggest `/list-projects` for full overview or `/review-projects` for deep review
5. Show task store stats: "Project #api-redesign has 4 ready, 2 in-progress, 1 waiting in the task store"

### When user wants to break down a project:
1. Help identify actionable tasks (don't invent — ask)
2. Suggest creating tasks with the project's `#tag` — they'll go to the task store
3. Use `/dump` for multiple tasks or `/add-task` for one at a time
4. Use `/refine` to break a task into subtasks with IDs
5. Remind about dependencies between tasks (reference by t-NNN IDs)

### When project seems stuck:
1. Check task store for project tasks: `/list-tasks --tag #project-tag`
2. Look for tasks in waiting or paused status — these might be blockers
3. Ask about blockers — don't assume
4. Suggest reviewing milestones or adjusting timeline
5. Offer `/update-project` to change status or add notes

## Project-Area Relationship

- A project can belong to an area: `**Area:** @home` or `**Area:** @work/engineering`
- Sub-areas work too: a project can link to `@work/engineering` specifically
- When creating a project, suggest linking to an area if relevant
- The area gives context; the project gives structure
- When a project completes, the area continues

## Tag System

- Projects use `#tags`: `#kitchen-reno`, `#tax-filing-2026`
- Areas use `@tags`: `@home`, `@work/engineering`
- Tasks get ONE tag type — if it has `#project-tag`, it doesn't also get `@area-tag`
- The project's area linkage provides the area context

## Templates

Available template locations:
- **Project-only templates** in `templates/projects/`:
  - default.md — Basic project structure
  - client-project.md — Client work with budget/invoicing
  - personal-goal.md — Personal goals with habits/rewards
- **Unified templates** in `templates/tasks/` (with `Creates project: yes`):
  - Can create a project AND tasks in one operation via `/new-from-template`

Suggest the appropriate template when creating a project. For projects with known workflows, unified templates save significant setup time.

## Progress Tracking

- Progress auto-calculates from completed vs total tasks with the project's `#tag` in the task store
- Task store provides accurate counts: ready, in-progress, waiting, paused, done
- Milestones track key checkpoints with dates
- `/review-projects` gives weekly analysis: on-track, at-risk, stalled, overdue

## Hard Rules

- **Don't invent project structure** — ask the user what they need
- **Don't assume scope** — let the user define goals and milestones
- **Suggest, don't create** — recommend `/new-project` rather than creating one silently
- **Respect existing tags** — don't reassign tasks between projects without asking
- **Reference task IDs** — when discussing project tasks, use their t-NNN IDs
