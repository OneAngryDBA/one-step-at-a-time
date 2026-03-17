---
name: project-management
description: This skill should be used when the user mentions projects, long-term goals, multi-task efforts, client work, launches, milestones, deadlines, or wants to track progress on larger initiatives. It also applies when the user describes something that sounds like a project (has multiple steps and an end goal) but has not created one yet, or asks about project status, progress, or planning. It should not be used for simple one-off tasks, daily routine questions, or chores/maintenance items (those belong to area-management).
---

# Project Management Skill

Help users manage projects effectively using OStaaT's project system.

## Recognizing Project-Scale Work

A project is appropriate when:
- There are **multiple tasks** toward a **single goal**
- There's a **target end date** or deadline
- It's **not recurring** (recurring → area, not project)
- It needs **progress tracking**
- Examples: "Redesign the website", "File 2026 taxes", "Plan vacation", "Launch new product"

A project is NOT appropriate when:
- It's a single task (just add it to today's todo)
- It's an ongoing responsibility (that's an area: @home, @health)
- It's a recurring chore (that's a maintenance item in an area)

## How to Help

### When user describes something that sounds like a project:
1. Recognize it: "That sounds like it could be a project — multiple steps with a clear goal"
2. Suggest: "Want to create a project for it? `/new-project` will walk you through it"
3. Don't force it — if they just want tasks, help with tasks

### When user asks about project status:
1. Read `PROJECTS.md`
2. Show relevant project details (progress, tasks, deadlines)
3. Suggest `/list-projects` for full overview or `/review-projects` for deep review

### When user wants to break down a project:
1. Help identify actionable tasks (don't invent — ask)
2. Suggest creating tasks with the project's `#tag`
3. Use `/dump` for multiple tasks or `/add-task` for one at a time
4. Remind about dependencies between tasks

### When project seems stuck:
1. Check for stalled indicators (no activity in 7+ days)
2. Ask about blockers — don't assume
3. Suggest reviewing milestones or adjusting timeline
4. Offer `/update-project` to change status or add notes

## Project-Area Relationship

- A project can belong to an area: `**Area:** @home`
- When creating a project, suggest linking to an area if relevant
- The area gives context; the project gives structure
- When a project completes, the area continues

## Tag System

- Projects use `#tags`: `#kitchen-reno`, `#tax-filing-2026`
- Areas use `@tags`: `@home`, `@finances`
- Tasks get ONE tag type — if it has `#project-tag`, it doesn't also get `@area-tag`
- The project's area linkage provides the area context

## Templates

Available in the plugin's `templates/projects/` directory:
- **default.md** — Basic project structure
- **client-project.md** — Client work with budget/invoicing
- **personal-goal.md** — Personal goals with habits/rewards

Suggest the appropriate template when creating a project.

## Progress Tracking

- Progress auto-calculates from completed vs total tasks with the project's `#tag`
- Milestones track key checkpoints with dates
- `/review-projects` gives weekly analysis: on-track, at-risk, stalled, overdue

## Hard Rules

- **Don't invent project structure** — ask the user what they need
- **Don't assume scope** — let the user define goals and milestones
- **Suggest, don't create** — recommend `/new-project` rather than creating one silently
- **Respect existing tags** — don't reassign tasks between projects without asking
