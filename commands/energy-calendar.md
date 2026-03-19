---
description: "View and manage your energy calendar — weekly defaults, daily overrides, capacity overview"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Manage Energy Calendar

**Mode: CO-PILOT**

**Prerequisite:** `energy.enabled` must be true in ostaat.json with at least one dimension configured. If not, inform the user and suggest running `/setup` or `/upgrade` to configure energy dimensions first.

## Usage

Can be invoked with arguments or interactively:
- `/energy-calendar` — show this week's calendar with today highlighted
- `/energy-calendar today` — show today's blocks and capacity
- `/energy-calendar week` — show full weekly defaults
- `/energy-calendar YYYY-MM-DD` — show a specific date (with overrides applied)
- `/energy-calendar edit` — interactive editing mode
- `/energy-calendar override YYYY-MM-DD` — add/edit override for a specific date
- `/energy-calendar capacity YYYY-MM-DD to YYYY-MM-DD` — show capacity summary for a date range

## Step 1: Load Energy Calendar

1. Read `ENERGY-CALENDAR.md` from `{dataDir}/`
2. If it doesn't exist, offer to create it: "No energy calendar found. Set one up now?"
   - If yes, go to Initial Setup flow
   - If no, exit

3. Parse weekly defaults and overrides
4. Read `energy.dimensions` from ostaat.json for validation

## View: Today

Show today's resolved energy blocks (weekly default + any overrides applied):

```
📊 Energy Calendar — Thursday, March 19, 2026

  09:00-12:00  🧠 Deep @work           (3.0 hrs)
  12:00-13:00  (break)
  13:00-14:30  🤝 Social @work          (1.5 hrs)
  14:30-17:00  ⚡ Admin @work           (2.5 hrs)
  18:00-19:00  ⚡ Admin @home           (1.0 hr)

  Total: 8.0 hrs across 4 blocks

  Summary by energy type:
  - 🧠 Deep: 3.0 hrs
  - 🤝 Social: 1.5 hrs
  - ⚡ Admin: 3.5 hrs (2.5 @work + 1.0 @home)
```

If Google Calendar MCP is available, overlay calendar events:
```
  Calendar conflicts:
  - 13:00-13:30  Team standup (reduces 🤝 Social by 30 min)
  - 15:00-15:30  Client call (reduces ⚡ Admin by 30 min)

  Adjusted availability: 7.0 hrs
```

## View: Week

Show the full weekly defaults with any overrides for this week:

```
📊 Energy Calendar — Week of March 17, 2026

Mon  │ 🧠 3.0h  🤝 1.5h  ⚡ 3.5h  🎨 1.0h │ 9.0 hrs
Tue  │ 🧠 3.0h  🤝 1.5h  ⚡ 3.5h           │ 8.0 hrs
Wed  │ 🧠 5.0h            ⚡ 3.0h  🎨 1.0h │ 9.0 hrs
Thu  │ 🧠 3.0h  🤝 1.5h  ⚡ 3.5h           │ 8.0 hrs ← today
Fri  │ 🧠 3.0h  🤝 2.0h  ⚡ 2.0h           │ 7.0 hrs
Sat  │                    ⚡ 2.0h  🎨 2.0h │ 4.0 hrs  (+ 🧠 1.0h @personal)
Sun  │ (rest day)                            │ 0.0 hrs

Week total: 45.0 hrs

Overrides this week:
- Thu Mar 20: 09:00-17:00 🤝 Social @work (all-day offsite) — replaces defaults
- Fri Mar 21: (day off)
```

## View: Capacity (date range)

Show available capacity by energy type and area for a date range:

```
📊 Capacity: March 19 – March 31 (13 days)

🧠 Deep:
  @work:     27.0 hrs (9 weekdays × 3.0h)
  @personal:  2.0 hrs (2 Saturdays × 1.0h)
  Total:     29.0 hrs

🤝 Social:
  @work:     10.5 hrs
  Total:     10.5 hrs

⚡ Admin:
  @work:     22.5 hrs
  @home:      7.0 hrs
  @home/maintenance: 4.0 hrs (2 Saturdays × 2.0h)
  Total:     33.5 hrs

🎨 Creative:
  @personal:  6.0 hrs
  Total:      6.0 hrs

Grand total: 79.0 hrs

Overrides reducing capacity:
- Mar 20: -🧠 Deep @work 3.0h, -⚡ Admin @work 2.5h (offsite replaces defaults)
- Mar 21: -all (day off)
Adjusted total: 70.5 hrs
```

## Action: Edit Weekly Defaults

Interactive editing of weekly defaults:

1. Show current weekly defaults
2. Ask: "Which day to edit?" (or "all weekdays" for bulk edit)
3. For the selected day, show current blocks
4. Options:
   - **Add block** — "Add a new time block"
     - Ask: start time, end time, energy type (from configured dimensions), area scope
     - Validate: no overlap with existing blocks, times are reasonable
   - **Edit block** — "Change an existing block"
     - Show numbered list of blocks, ask which to modify
     - Can change time range, energy type, or area scope
   - **Remove block** — "Remove a block"
     - Show numbered list, confirm deletion
   - **Copy from another day** — "Copy Monday's blocks to Tuesday"
   - **Done** — Save changes

5. Write updated ENERGY-CALENDAR.md

## Action: Add Override

Add a date-specific override:

1. Ask for date (or accept from argument)
2. Show what the weekly default would be for that day
3. Ask: "Replace the entire day, add a block, or mark as day off?"
   - **Replace** → define new blocks for that day (full schedule)
   - **Add block** → prefix with `+`, adds to existing defaults
     - Example: `+19:00-21:00 🧠 Deep @work (evening catchup)`
   - **Day off** → `(day off)` — clears all blocks
   - **Date range** → ask for end date, applies to all days in range

4. Add to Overrides section in ENERGY-CALENDAR.md (sorted by date)

## Action: Remove Override

1. Show existing overrides
2. Ask which to remove
3. Remove from ENERGY-CALENDAR.md (day reverts to weekly default)

## Initial Setup Flow

If no ENERGY-CALENDAR.md exists:

1. Read `energy.dimensions` from ostaat.json for available energy types
2. Ask: "Let's set up your energy calendar. I'll ask about your typical week."

3. **Weekday template:**
   - "What time do you typically start work?"
   - "What time do you finish?"
   - "When's your lunch break?"
   - "Which energy type fits your morning best?" (show configured options)
   - "Which fits your afternoon?"
   - "Do you have dedicated time blocks for specific energy types?"
   - Build weekday template from answers

4. **Individual day adjustments:**
   - "Any days that differ from this pattern?"
   - Common: "Friday afternoons are lighter", "Wednesday is no-meeting day"

5. **Weekend:**
   - "Do you do any structured work on weekends?"
   - If yes, ask about Saturday and Sunday blocks
   - If no, mark as rest days

6. **Area scoping:**
   - "Which areas do these blocks belong to?"
   - For work blocks: default to `@work` (can refine to sub-areas)
   - For personal blocks: ask which area

7. **Evening blocks:**
   - "Do you do personal tasks in the evening?"
   - If yes, add evening blocks with home/personal area scope

8. Write ENERGY-CALENDAR.md
9. Show the complete calendar for confirmation

## Git Commit

If `git.autoCommit` enabled:
```
Update energy calendar

- {{description of changes}}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Philosophy:** The energy calendar is a planning aid, not a taskmaster. It helps you understand your capacity and make realistic plans. Edit it freely as your rhythms change.
