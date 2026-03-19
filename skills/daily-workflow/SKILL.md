---
name: daily-workflow
description: This skill should be used when the user starts their day, asks about planning their day, mentions morning routine or end-of-day review, says "what should I focus on", "how's my day looking", "I'm done for today", "wrapping up", or discusses daily planning, prioritization, or time management. It also applies when the user seems to have too many tasks or asks about what to tackle first. It should not be used for general conversation, coding questions, or one-off task mentions that don't involve daily planning.
---

# Daily Workflow Skill

Guide users through effective daily task management using the task store as the source of truth.

## Daily Rhythm

### Morning → `/start-day`
When user starts their day or asks about planning:
1. Suggest `/start-day` if today's files don't exist yet
2. If files exist, help them review what's on the plate
3. Surface key priorities from the task store: in-progress tasks, overdue items, due dates, area reminders, lead-time-triggered tasks
4. Help them adjust today's focus if needed

### During the Day → `/dump`, `/add-task`
When user mentions new tasks mid-day:
1. Capture quickly — suggest `/dump` for multiple items, `/add-task` for one
2. Tasks go to the task store first, optionally added to today's focus
3. Help prioritize against existing tasks in the store
4. If load is heavy, suggest deferring lower-priority items back to ready

### End of Day → `/review-day`
When user is wrapping up:
1. Suggest `/review-day` to mark completions in the task store
2. Help them feel good about what they accomplished (without being cheesy)
3. Surface tomorrow's upcoming items from the store
4. Don't judge incomplete tasks — they stay in the store, no rolling forward needed

## Prioritization Help

When a user asks what to do first:

1. **Read task store** — `tasks/in-progress.md` for today's focus, `tasks/ready.md` for backlog
2. **Prioritize by**:
   - 📋 Overdue obligations first (real consequences — check AREAS.md)
   - 🔴 High priority + ⏰ Now items
   - Items with due dates approaching (📅 field) or lead-time triggered
   - Dependencies (unblock other tasks — check `Dependencies:` fields)
   - Quick wins (🔧 < 15 min, builds momentum)
3. **Present top 3** focus items with their task IDs
4. **Suggest** moving non-essential items back to ready via `/update-task`

## Task Status Management

Help users transition tasks between states:
- "I'm blocked on X" → suggest `/update-task t-NNN waiting`
- "I'll do that later" → suggest `/update-task t-NNN paused` or move to ready
- "I finished X" → note it for `/review-day` or suggest immediate update
- "What's in my inbox?" → suggest `/list-tasks --inbox` or `/list-tasks --status inbox`

## Load Awareness

If the user's task store shows many in-progress tasks:
- Count tasks and total 🔧 time from `tasks/in-progress.md`
- Compare against thresholds from `ostaat.json` (areas.loadThresholds)
- If heavy, **present data neutrally**: "You have 12 tasks in progress totaling ~6 hours"
- Offer to triage: move some back to ready, pause, or break into smaller subtasks via `/refine`
- **Never say** "you're overwhelmed" or assume emotional state

## Time-Aware Suggestions

- **Before 10am**: Suggest `/start-day` if not done
- **Mid-day**: If user mentions tasks, help add/prioritize via task store
- **After 4pm**: If user seems to be wrapping up, suggest `/review-day`
- **Weekend**: Lighter touch — suggest `/review-areas` if not reviewed in the past 7 days, or `/review-projects` for weekly check-in

## Integration Points

- Read `tasks/in-progress.md` to assess today's focus tasks
- Read `tasks/ready.md` to know what's available
- Read `tasks/waiting.md` for blocked items
- Read `tasks/INDEX.md` for quick lookups
- Read `YYYY-MM-DD-todo.md` as the daily working view
- Read `YYYY-MM-DD-finished.md` to see what's been accomplished
- Read `AREAS.md` for recurring items due today and load context
- Read `PROJECTS.md` for active project deadlines
- Check `scheduled/` for pre-planned future tasks
- Reference `ostaat.json` for thresholds and preferences

## Hard Rules

- **Never assume emotional states** — present data, let user decide
- **Don't be pushy** — suggest commands, don't demand them
- **Celebrate progress** without being patronizing
- **No judgment** on incomplete tasks — they persist in the store
- **One suggestion at a time** — don't dump a list of commands to run
- **Task store is authoritative** — always reference task IDs when discussing tasks
