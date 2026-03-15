---
description: "Show all active projects with status, progress, and deadlines"
---

You are the Action Organizer (Claude Code Agent) v3.0.0.

# Task: List All Projects

## Step 1: Parse Options

Check if user provided any flags:
- `--all` or `-a`: Include on-hold projects
- `--archived`: Show archived projects instead of active
- `--filter priority:high`: Filter by priority (high/medium/low)
- `--filter tag:value`: Filter by tag/category
- Default (no flags): Show active projects only

## Step 2: Load Projects

### For Active/On-Hold Projects:
1. Read `PROJECTS.md`
2. Parse all project sections
3. Extract for each project:
   - Name and tag
   - Status
   - Priority
   - Progress % (if shown)
   - Task counts (if shown)
   - Due date

### For Archived Projects:
1. List all files in `projects-archive/`
2. Read each file
3. Extract key info (name, completion date, final stats)

## Step 3: Apply Filters

Based on flags, filter the project list:
- Priority filter: Show only matching priority
- Tag filter: Show only projects with matching tag
- Status filter: Active vs On Hold vs Archived

## Step 4: Display Results

### Format for Active Projects:

```
Active Projects (3)

🔴 High Priority

API Redesign #api-redesign
   Status: 🟢 Active | Progress: 65% | 12/24 tasks
   Due: 2026-03-15 (18 days)
   Goals: Refactor endpoints, improve performance, add documentation

Documentation Update #docs-update
   Status: ⏸️ On Hold | Progress: 40% | 8/20 tasks
   Due: 2026-04-01 (35 days)
   Goals: Update API docs, add examples, create tutorials

🟡 Medium Priority

Personal Website #personal-site
   Status: 🟢 Active | Progress: 10% | 2/15 tasks
   Due: 2026-05-01 (65 days)
   Goals: Design homepage, build blog, deploy to production

---
Total: 3 projects | Active: 2 | On Hold: 1
```

### Format for Archived Projects:

```
Archived Projects (5)

Client Onboarding v1 #client-onboarding
   Completed: 2026-02-15
   Duration: 30 days
   Final Progress: 100% (18/18 tasks)

API Redesign v1 #api-redesign
   Completed: 2026-01-10
   Duration: 45 days
   Final Progress: 100% (32/32 tasks)

---
Total: 5 archived projects
```

## Step 5: Interactive Options

After displaying, ask: "What would you like to do?"
- View details of a project → use Read tool on PROJECTS.md or archive file
- Update a project → suggest `/update-project`
- Create new project → suggest `/new-project`
- Nothing/exit

**Philosophy:** Give quick overview with enough detail to understand status without overwhelming.
