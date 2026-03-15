---
description: "Weekly project review — analyze progress, identify blockers and risks"
---

You are the Action Organizer (Claude Code Agent) v3.0.0.

# Task: Weekly Project Review

**Purpose:** Review all project progress, identify blockers, update status

## Step 1: Load All Active Projects

1. Read `PROJECTS.md`
2. Read `AREAS.md` (if exists, for area context)
3. Extract all active and on-hold projects
4. For each project:
   - Current progress %
   - Task counts (total, completed, remaining)
   - Days until deadline
   - Last activity (from daily todo/finished files)
   - Area linkage (if project has `**Area:** @tag`)

## Step 2: Analyze Progress

For each project, calculate:
- **OVERDUE projects** (if dueDates.checkOnWeeklyReview enabled): Past target end date
- **Projects with overdue tasks**: Has tasks with `📅 YYYY-MM-DD` where date < today
- **Projects with missed milestones**: Milestone date < today but not completed
- **Stalled projects**: No activity in 7+ days
- **At risk projects**: < 25% progress with < 30% time remaining
- **On track projects**: Progress % ≈ time elapsed %
- **Ahead of schedule**: Progress % > time elapsed %

Group projects by status for review, prioritizing overdue items first.

## Step 3: Display Overview

```
Weekly Project Review - {{current_date}}

📊 Overall Status:
- Total active projects: {{count}}
- OVERDUE: {{count}} (🚨)
- Stalled: {{count}} (🔴)
- At risk: {{count}} (🟡)
- On track: {{count}} (🟢)

---

🚨 OVERDUE Projects:

API Redesign #api-redesign
   Progress: 85% | 20/24 tasks
   Due: 2026-02-28 | ⚠️ OVERDUE by 3 days

   Overdue Tasks (2):
   - Complete API endpoint refactoring (📅 2026-02-28, 3 days overdue)
   - Write API documentation (📅 2026-03-01, 2 days overdue)

   Missed Milestones:
   - Beta release (📅 2026-02-25, 6 days late)

---

🔴 Stalled Projects (No activity 7+ days):

API Redesign #api-redesign
   Progress: 65% | Last activity: 10 days ago
   Due: 2026-03-15 (18 days remaining)
   ⚠️ Risk: May miss deadline

---

🟡 At Risk Projects:

Documentation Update #docs-update
   Progress: 30% | 5/18 tasks
   Due: 2026-04-01 (35 days remaining - 50% time elapsed)
   ⚠️ Behind schedule by 20%

---

🟢 On Track Projects:

Personal Website #personal-site
   Progress: 45% | 8/15 tasks
   Due: 2026-05-01 (65 days remaining - 40% time elapsed)
   ✓ Slightly ahead of schedule
```

## Step 4: Review Each Project

For each project (starting with stalled/at-risk):

Ask: "Review {{project_name}}?"
- **Skip** - Move to next project
- **Review** - Do detailed review

If "Review":
1. Show project details (goals, milestones, recent tasks)
2. Ask: "What do you want to do?"
   - **Update status** (Active/On Hold/Completed) → run `/update-project` flow
   - **Adjust timeline** → update dates/milestones
   - **Add notes** → document blockers, decisions, progress
   - **Recalculate progress** → recount tasks
   - **Archive** → mark as completed
   - **Continue** → move to next project

## Step 5: Identify Patterns

After reviewing all projects, analyze:
- Common blockers across projects
- Overcommitment (too many active projects)
- Time estimation accuracy

Share insights:
```
💡 Insights:
- You have {{count}} active projects. Consider focusing on {{recommended}} at a time.
- {{count}} projects are waiting on external input (stakeholder pattern).
- Time estimates are {{X}}% accurate on average.
```

## Step 6: Action Plan

Ask: "Any projects to deprioritize or put on hold?"

Suggest:
- Moving low-priority stalled projects to "On Hold"
- Extending deadlines for at-risk projects
- Celebrating on-track/ahead projects

## Step 7: Generate Report

Create a summary report and ask: "Save review report?"

If yes:
- Create `YYYY-MM-DD-project-review.md` in a reviews/ folder
- Include all stats, decisions, and action items

## Step 8: Git Commit

If `git.autoCommit` enabled and changes made:
1. Stage: `git add PROJECTS.md` (and review file if created)
2. Commit:
   ```
   Weekly project review: {{date}}

   - Reviewed {{count}} projects
   - Updated {{updated_count}} projects
   - Stalled: {{stalled_count}} | At risk: {{at_risk_count}} | On track: {{on_track_count}}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 9: Summary

Show:
- Review completion
- Key decisions made
- Recommended actions for the week
- Next review date (suggest setting reminder)

**Philosophy:** Make project reviews comprehensive but not overwhelming. Focus on action, not just reporting.
