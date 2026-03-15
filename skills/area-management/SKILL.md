---
name: area-management
description: This skill should be used when managing areas of responsibility, recurring items, chores, habits, maintenance, and obligations using the Action Organizer PARA system. It applies when the user mentions chores, household tasks, habits, routines, recurring responsibilities, bills, maintenance items, life management, "I keep forgetting to", "I need to remember to", health habits, personal upkeep, or anything that repeats on a schedule without a defined end goal. It also applies when the user asks about their areas, recurring items, or what is overdue. It should not be used for one-off tasks (use task-organization), project work with deadlines (use project-management), or daily planning questions (use daily-workflow).
---

# Area Management Skill

Help users manage their areas of responsibility — the ongoing parts of life that need regular attention but never "complete."

## Three-Tier System

### 🧘 Habits — Daily routines, always soft
- Brush teeth, exercise, meditate, eat breakfast, journal
- Tracked by frequency (daily, X times per week)
- Missing one is fine — gentle nudge at most
- No consequences

### 🔄 Maintenance — Periodic chores, soft deadlines
- Vacuum, mow lawn, clean gutters, oil change, laundry
- Tracked by cadence (weekly, monthly, seasonal)
- Missing one isn't critical but builds up over time
- No formal consequences, but things get worse if neglected

### 📋 Obligations — Hard deadlines with consequences
- File taxes, pay bills, renew license, insurance, registrations
- Tracked by hard due dates
- Missing has real consequences (fines, penalties, damage)
- Consequences are documented and surface when overdue

## Recognizing Area Items in Conversation

| User says | Likely tier | Why |
|-----------|------------|-----|
| "I should exercise more" | 🧘 Habit | Personal routine, no deadline |
| "I need to vacuum" | 🔄 Maintenance | Periodic chore, soft |
| "Rent is due Friday" | 📋 Obligation | Hard deadline, consequence |
| "I keep forgetting to water the plants" | 🔄 Maintenance | Periodic, "keep forgetting" = needs tracking |
| "My car registration expires next month" | 📋 Obligation | Hard deadline, legal consequence |
| "I should probably clean the gutters" | 🔄 Maintenance | Seasonal, no hard deadline |

## How to Help

### When user mentions a recurring responsibility:
1. Recognize it: "That sounds like a recurring item for your @home area"
2. Check if the area exists in `AREAS.md`
3. If area exists: suggest adding the item via `/update-area`
4. If area doesn't exist: suggest creating it via `/new-area`
5. Help classify the tier (habit/maintenance/obligation)

### When user says "I keep forgetting to...":
This is a strong signal they need tracking:
1. Identify the right area
2. Classify the tier
3. Set a cadence
4. Add it so the system will remind them

### When user asks about overdue or due items:
1. Read `AREAS.md`
2. Show items by status (overdue first, then due today, then nudging)
3. For 📋 Obligations, show consequences
4. Suggest `/list-areas --due` for a filtered view

### When user wants to discuss an obligation:
1. Acknowledge the hard deadline
2. Show consequences if overdue
3. If it's complex enough, suggest promoting to a project
4. Help them plan when to tackle it

## Consequence Tracking

For 📋 Obligations, consequences are stored as:
```
⚠️ If missed: {{immediate consequence}}. After {{time}}: {{escalation}}
```

When surfacing overdue obligations, always show the consequence line. Examples:
- `⚠️ If missed: Late fee ($29). After 30 days: interest + credit score impact`
- `⚠️ If missed: Late filing penalty + interest. After 60 days: failure-to-file penalty (5%/month)`

## Promote to Project

Some recurring items become complex when it's time to do them:
- "File taxes" → needs subtasks: gather docs, choose method, fill forms, review, submit
- "Annual physical" → might need follow-up appointments

Recognize when this is happening and suggest: "This seems like it needs its own plan. Want to promote it to a project? The recurring item stays in your area for next time."

## Load Awareness

If a user has many overdue or nudging items:
- Present the count neutrally
- Offer to help triage (pause some, adjust cadences, defer)
- Never say "you're overwhelmed" — just show the data
- Suggest `/review-areas` for a comprehensive check-in

## Hard Rules

- **Never suggest areas the user didn't mention** — don't invent responsibilities
- **Never assume what someone should be doing** — only track what they ask for
- **Present data, not judgment** — "3 items overdue" not "you're falling behind"
- **Respect the tiers** — don't escalate a habit to an obligation without the user asking
- **Nudge, don't nag** — surface gently, let them decide
