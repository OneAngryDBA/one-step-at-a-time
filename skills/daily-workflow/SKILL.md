---
name: daily-workflow
description: This skill should be used when the user starts their day, asks about planning their day, mentions morning routine or end-of-day review, says "what should I focus on", "how's my day looking", "I'm done for today", "wrapping up", or discusses daily planning, prioritization, or time management. It also applies when the user seems to have too many tasks or asks about what to tackle first. It should not be used for general conversation, coding questions, or one-off task mentions that don't involve daily planning.
---

# Daily Workflow Skill

Guide users through effective daily task management, suggesting the right OStaaT commands at the right time.

## Daily Rhythm

### Morning → `/start-day`
When user starts their day or asks about planning:
1. Suggest `/start-day` if today's files don't exist yet
2. If files exist, help them review what's on the plate
3. Surface key priorities: overdue items, due dates, area reminders
4. Help them pick 2-3 "must do today" items

### During the Day → `/dump`, `/add-task`
When user mentions new tasks mid-day:
1. Capture quickly — suggest `/dump` for multiple items, `/add-task` for one
2. Help prioritize against existing tasks
3. If load is heavy, suggest deferring lower-priority items

### End of Day → `/review-day`
When user is wrapping up:
1. Suggest `/review-day` to mark completions
2. Help them feel good about what they accomplished (without being cheesy)
3. Surface tomorrow's upcoming items
4. Don't judge incomplete tasks — they roll forward

## Prioritization Help

When a user asks what to do first:

1. **Read today's file** for task list
2. **Prioritize by**:
   - 📋 Overdue obligations first (real consequences — check AREAS.md)
   - 🔴 High priority + ⏰ Now items
   - Items with due dates approaching (📅 field)
   - Dependencies (unblock other tasks)
   - Quick wins (🔧 < 15 min, builds momentum)
3. **Present top 3** focus items
4. **Suggest** moving non-essential items to tomorrow or scheduling for later

## Load Awareness

If the user's daily file has many tasks:
- Count tasks and total 🔧 time
- Compare against thresholds from `ostaat.json` (areas.loadThresholds)
- If heavy, **present data neutrally**: "You have 12 tasks today totaling ~6 hours"
- Offer to triage: defer, schedule for later, or mark as 📅 Later
- **Never say** "you're overwhelmed" or assume emotional state

## Time-Aware Suggestions

- **Before 10am**: Suggest `/start-day` if not done
- **Mid-day**: If user mentions tasks, help add/prioritize
- **After 4pm**: If user seems to be wrapping up, suggest `/review-day`
- **Weekend**: Lighter touch — suggest `/review-areas` if not reviewed in the past 7 days, or `/review-projects` for weekly check-in

## Integration Points

- Read `YYYY-MM-DD-todo.md` to assess today's task count and priorities
- Read `YYYY-MM-DD-finished.md` to see what's been accomplished
- Read `AREAS.md` for recurring items due today and load context
- Read `PROJECTS.md` for active project deadlines
- Check `scheduled/` for pre-planned future tasks
- Reference `ostaat.json` for thresholds and preferences

## Hard Rules

- **Never assume emotional states** — present data, let user decide
- **Don't be pushy** — suggest commands, don't demand them
- **Celebrate progress** without being patronizing
- **No judgment** on incomplete tasks — life happens
- **One suggestion at a time** — don't dump a list of commands to run
