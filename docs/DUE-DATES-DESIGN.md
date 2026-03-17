# Due Date Tracking System Design

**Version:** 1.0.0
**Status:** Implemented

---

## Overview

Add due date tracking to tasks and projects with automatic checks for overdue items at day start/end and in weekly reviews.

---

## Task Due Date Format

### Syntax

Tasks can include an optional due date using the 📅 emoji followed by the date:

```markdown
- 🔴 High ⏰ Now 🔧 45 min 📅 2026-03-05 #project-tag
  Complete API endpoint refactoring
```

### Date Format Options

Support multiple formats for ease of use:
- **Full date:** `📅 2026-03-05` or `📅 2026-03-05`
- **Relative:** `📅 tomorrow`, `📅 today`, `📅 next week`
- **Natural:** `📅 Mon`, `📅 Friday`, `📅 end of week`

System normalizes all to `YYYY-MM-DD` format when saving.

### Placement

Due date appears after timing and time estimate, before project tag:
```
Priority Timing Time DueDate ProjectTag
🔴 High ⏰ Now 🔧 45 min 📅 2026-03-05 #api-redesign
```

---

## Project Due Dates

Projects already have timeline fields with target end dates. These are used for:
- Progress tracking
- At-risk calculations
- Milestone tracking

Tasks inherit urgency from project deadlines but can have their own due dates.

---

## Feature 1: Overdue Check at /start-day

### Behavior

When running `/start-day`, after rolling forward tasks:

1. **Scan for overdue tasks:**
   - Check today's todo file (rolled forward tasks)
   - Look for tasks with `📅 YYYY-MM-DD` where date < today

2. **Display overdue section:**
   ```
   ⚠️ Overdue Tasks (3):

   - 🔴 High 🔧 45 min 📅 2026-02-28 (2 days overdue) #api-redesign
     Complete API endpoint refactoring

   - 🟡 Medium 🔧 30 min 📅 2026-03-01 (1 day overdue)
     Update documentation

   - 🟢 Low 🔧 15 min 📅 2026-02-25 (5 days overdue) #client-project
     Send follow-up email
   ```

3. **Ask for action:**
   ```
   What would you like to do with overdue tasks?
   - Reschedule all → asks for new date
   - Reschedule individually → one at a time
   - Mark for today (⏰ Now) → updates timing
   - Review later → skip for now
   ```

4. **Update tasks based on choice**

### Implementation in /start-day

Add new step after rolling forward tasks:

```markdown
**Step X: Check for Overdue Tasks**

1. Scan rolled-forward tasks for due dates
2. Identify tasks where due date < today
3. If overdue tasks found:
   - Display list with days overdue
   - Group by priority
   - Ask for action (reschedule/mark urgent/skip)
4. Update tasks based on user choice
```

---

## Feature 2: Upcoming Due Dates at /review-day

### Behavior

When running `/review-day`, after moving completed tasks:

1. **Scan remaining tasks for upcoming due dates:**
   - Tomorrow (date = today + 1)
   - This week (date <= end of week)
   - Next week (date in next 7-14 days)

2. **Display upcoming section:**
   ```
   📅 Upcoming Due Dates:

   Tomorrow (3):
   - 🔴 High 🔧 60 min 📅 2026-03-03 #api-redesign
     Deploy to production
   - 🟡 Medium 🔧 30 min 📅 2026-03-03
     Client meeting prep
   - 🟢 Low 🔧 15 min 📅 2026-03-03
     Update invoice

   This Week (2):
   - 🔴 High 🔧 90 min 📅 2026-03-07 #docs-update
     Finish documentation overhaul
   - 🟡 Medium 🔧 45 min 📅 2026-03-06
     Code review session
   ```

3. **Ask for planning:**
   ```
   Ready for tomorrow's deadlines?
   - Yes, all set
   - Need to reschedule some → which ones?
   - Need to break down tasks → run /refine
   ```

### Implementation in /review-day

Add new step after moving completed tasks:

```markdown
**Step X: Show Upcoming Due Dates**

1. Scan remaining tasks in today's todo for due dates
2. Group by timeframe:
   - Tomorrow (date = today + 1)
   - This week (date <= end of current week)
   - Next week (date in next 7-14 days)
3. Display grouped list
4. Ask if user wants to reschedule or refine any
5. Offer to add reminders or adjust priorities
```

---

## Feature 3: Overdue Items in Weekly Project Review

### Behavior

When running `/review-projects`, include overdue analysis:

1. **For each project, check:**
   - Tasks with due dates that have passed
   - Milestones that were missed
   - Projects past target end date

2. **Display in project review:**
   ```
   🔴 API Redesign #api-redesign
      Progress: 65% | 12/24 tasks | Due: 2026-03-15
      ⚠️ OVERDUE: Target date passed by 3 days

      Overdue Tasks (2):
      - Complete API endpoint refactoring (📅 2026-02-28, 7 days overdue)
      - Write API documentation (📅 2026-03-10, 2 days overdue)

      Missed Milestones:
      - Beta release (📅 2026-03-01, 4 days late)
   ```

3. **Ask for each project with overdue items:**
   ```
   Action for API Redesign?
   - Extend deadline → set new target date
   - Mark tasks as completed → if actually done
   - Reschedule tasks → update individual due dates
   - Add notes → document blockers/reasons
   - Put on hold → if blocked
   ```

### Implementation in /review-projects

Update the analysis section:

```markdown
**Step 2: Analyze Progress**

For each project, calculate:
- Stalled projects (no activity in 7+ days)
- At risk projects (< 25% progress with < 30% time remaining)
- **OVERDUE projects:** Past target end date
- **Projects with overdue tasks:** Has tasks with due dates < today
- **Projects with missed milestones:** Milestone date < today but not completed
- On track projects

Group by status, prioritizing overdue items first.
```

---

## Due Date Intelligence

### Auto-Suggestions

When a task has no due date but related info exists:
- Project has deadline → suggest task due = project due - buffer
- Task has "urgent" or "ASAP" → suggest today or tomorrow
- Task says "by Friday" → extract and suggest date

### Warnings

Show warnings when:
- Task due date is after project due date
- Multiple high-priority tasks due same day
- Task estimated time > remaining time to deadline
- Task has dependencies that aren't complete yet

---

## Configuration

Add to `todo-config.json`:

```json
"dueDates": {
  "enabled": true,
  "checkOnStartDay": true,
  "checkOnReviewDay": true,
  "checkOnWeeklyReview": true,
  "warnings": {
    "taskAfterProject": true,
    "multipleHighPrioritySameDay": true,
    "insufficientTime": true
  },
  "autoSuggest": true,
  "bufferDays": 2
}
```

---

## Task Format Update

### Before
```markdown
- 🔴 High ⏰ Now 🔧 45 min #project-tag
  Complete API endpoint refactoring
  Dependencies: Database migration
  Context: See API design doc
```

### After (with due date)
```markdown
- 🔴 High ⏰ Now 🔧 45 min 📅 2026-03-05 #project-tag
  Complete API endpoint refactoring
  Dependencies: Database migration
  Context: See API design doc
  ⚠️ Due in 3 days
```

### Auto-Generated Status Lines

System can optionally add status lines:
- `⚠️ OVERDUE by 2 days`
- `⚠️ Due today`
- `📅 Due tomorrow`
- `📅 Due in 3 days`
- `📅 Due next week`

---

## Commands Update Summary

### /start-day
- **New:** Check overdue tasks
- **New:** Offer rescheduling/reprioritization
- **New:** Mark overdue as urgent option

### /review-day
- **New:** Show upcoming due dates (tomorrow, this week)
- **New:** Planning prompt for upcoming deadlines
- **New:** Suggest task breakdown if needed

### /review-projects
- **New:** Show overdue projects
- **New:** Show overdue tasks per project
- **New:** Show missed milestones
- **New:** Action prompts for each overdue item

### /dump & /add-task
- **New:** Ask for due date (optional)
- **New:** Auto-suggest based on context
- **New:** Validate against project deadlines

### /refine
- **New:** Option to add/update due dates
- **New:** Reschedule due dates

---

## Visual Indicators

### Priority by Due Date

Tasks can have computed urgency:
- 🔴 High + Overdue = 🚨 CRITICAL
- 🔴 High + Due today = ⚡ URGENT
- 🟡 Medium + Overdue = 🔴 High
- 🟢 Low + Overdue = 🟡 Medium

### Display Format

```markdown
🚨 CRITICAL: 🔴 High 🔧 45 min 📅 2026-02-28 (OVERDUE 2 days) #api-redesign
⚡ URGENT: 🔴 High 🔧 30 min 📅 2026-03-02 (DUE TODAY) #client-work
```

---

## Future Enhancements

- Calendar integration: Show due dates on calendar
- Notifications: Remind day before due date
- Recurring due dates: Weekly tasks, monthly reviews
- Team due dates: If tasks assigned to others
- Smart rescheduling: Suggest new dates based on capacity
- Due date history: Track how often things are rescheduled

---

## Implementation Order

1. ✅ Design complete (this document)
2. Update task format and parsing
3. Update /start-day with overdue check
4. Update /review-day with upcoming check
5. Update /review-projects with overdue analysis
6. Update /dump and /add-task for due date input
7. Add config section for due dates
8. Update documentation (README, task format)
9. Test end-to-end workflow
10. Git commit all changes

---

**Ready to implement!**
