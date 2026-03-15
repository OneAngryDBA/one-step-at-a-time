---
description: "Restore an archived project back to active status"
---

You are the Action Organizer (Claude Code Agent) v3.0.0.

# Task: Reopen Archived Project

**Purpose:** Pull an archived project back to active status

## Step 1: List Archived Projects

1. List all files in `projects-archive/`
2. Read each file to extract:
   - Project name
   - Completion date
   - Final stats (tasks, duration)
   - Tags
3. Display to user:
```
Archived Projects:

1. Client Onboarding #client-onboarding
   Completed: 2026-02-15 (10 days ago)
   Duration: 30 days | 18/18 tasks

2. API Redesign #api-redesign
   Completed: 2026-01-10 (45 days ago)
   Duration: 45 days | 32/32 tasks

3. Personal Website v1 #personal-site
   Completed: 2025-12-20 (67 days ago)
   Duration: 60 days | 28/28 tasks
```

## Step 2: Select Project

Ask: "Which project to reopen?"

User selects by number or name.

## Step 3: Reopen Options

Ask: "How to reopen?"
- **Fresh start** - Reset dates, progress, keep goals
- **Continue where left off** - Keep all data, just change status to Active
- **Create new iteration** - Start fresh but reference old one

## Step 4: Configure Reopened Project

### If "Fresh start":
1. Ask: "New start date?" (default: today)
2. Ask: "New target end date?"
3. Reset progress to 0%
4. Reset task counts to 0
5. Clear completed tasks list
6. Keep goals, stakeholders, resources

### If "Continue where left off":
1. Ask: "Extend deadline? Current was {{old_end_date}}"
2. Keep all existing data
3. Just change status from Completed → Active

### If "Create new iteration":
1. Ask: "What's different in this iteration?"
2. Keep old project in archive
3. Create new project with "(v2)" or next number in name
4. Reference previous iteration in notes

## Step 5: Handle Archived Copy

Ask: "Keep archived copy or remove?"
- **Keep**: Archived file stays (useful for reference, especially with iterations)
- **Remove**: Delete from archive
- **Rename**: If keeping and creating new iteration, rename old one (add "(v1)" if not present)

## Step 6: Add to PROJECTS.md

1. Add project under appropriate priority section
2. Status: 🟢 Active
3. Include all project details
4. Add note: "Reopened from archive on {{date}}"

## Step 7: Git Commit

If `git.autoCommit` enabled:
1. Stage changes: `git add PROJECTS.md projects-archive/`
2. Commit:
   ```
   Reopen project: {{project_name}}

   - Reopened from archive (completed {{completion_date}})
   - Mode: {{reopen_mode}}
   - New target date: {{new_end_date}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 8: Summary

Show:
- Project details
- How it was reopened
- Status of archived copy
- Suggested next action: "Add tasks with {{project_tag}} to get started"

**Philosophy:** Make it easy to restart or iterate on completed projects. Support learning from past work.
