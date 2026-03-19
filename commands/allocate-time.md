---
description: "Schedule tasks against energy blocks and Google Calendar availability"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Allocate Time — Energy-Aware Scheduling

**Mode: CO-PILOT**

## Usage

- `/allocate-time` — schedule today's focus tasks into energy blocks
- `/allocate-time tomorrow` — plan tomorrow's blocks
- `/allocate-time YYYY-MM-DD` — plan a specific day
- `/allocate-time week` — plan the upcoming week (capacity overview)

## Step 1: Load Tasks

1. Read today's focus from `tasks/in-progress.md` (or target date's tasks)
2. If planning tomorrow/future, read from `tasks/ready.md` for candidate tasks
3. For each task, note: energy label, active time (🔧), area tag, dependencies

## Step 2: Load Energy Blocks

1. Read `ENERGY-CALENDAR.md`
2. Resolve the target date: apply weekly default + any overrides
3. Parse into available blocks with energy type, area scope, and duration

### If Google Calendar MCP available:
4. Fetch calendar events for the target date
5. Subtract calendar events from energy blocks (meetings carve out time):
   ```
   Block: 09:00-12:00 🧠 Deep @work (180 min)
   Calendar: 10:00-10:30 Team standup
   Result: 09:00-10:00 🧠 Deep @work (60 min) + 10:30-12:00 🧠 Deep @work (90 min)
   ```
6. Show calendar overlay:
   ```
   📅 Calendar events today:
   - 10:00-10:30 Team standup
   - 13:00-14:00 Client meeting
   - 15:30-16:00 Code review
   ```

### If no MCP:
4. Use energy blocks as-is (user can manually note conflicts)

## Step 3: Match Tasks to Blocks

For each task, find compatible blocks using matching rules:
1. **Energy type match:** task's energy label matches block's energy type
2. **Area scope match:** block's area is equal to or an ancestor of the task's area tag
3. **Capacity:** block has enough remaining time for the task

### Scheduling algorithm:
1. Sort tasks by priority (🔴 → 🟡 → 🟢), then timing (⏰ → ⏭️ → 📅)
2. For each task:
   a. Find the earliest compatible block with enough time
   b. If task has dependencies, ensure it's scheduled after them
   c. Assign the task to the block, subtract its time from available capacity
   d. If task doesn't fit in any single block, split across blocks (note the split)

3. Track unscheduled tasks (no matching block or insufficient capacity)

## Step 4: Present Schedule

```
📅 Suggested Schedule — Thursday, March 19, 2026

🧠 Deep @work (09:00-12:00, 180 min available):
  09:00-09:45  [t-017] 🔴 🔧 45 min — API refactoring #api-redesign
  09:45-10:00  (15 min buffer)
  10:00-10:30  ── Team standup ── (calendar)
  10:30-11:00  [t-032] 🔧 30 min — Reconcile accounts #monthly-close
  11:00-11:30  [t-040] 🔴 🔧 30 min — Quarterly review prep
  11:30-12:00  (30 min remaining)

🤝 Social @work (13:00-14:30, 90 min available):
  13:00-14:00  ── Client meeting ── (calendar)
  14:00-14:30  (30 min remaining — no social tasks selected)

⚡ Admin @work (14:30-17:00, 150 min available):
  14:30-15:00  [t-031] 🔧 30 min — Pull bank statements #monthly-close
  15:00-15:20  [t-033] 🔧 20 min — Generate reports #monthly-close
  15:20-15:30  ── Code review ── (calendar, 15:30-16:00)
  16:00-16:15  [t-034] 🔧 15 min — Submit to accounting #monthly-close
  16:15-17:00  [t-035] 🔧 45 min — Update docs (partial, 45/60 min)
  ⚠️ [t-035] needs 15 more min — continues tomorrow or extend block

⚡ Admin @home (18:00-19:00, 60 min available):
  18:00-18:20  [t-048] 🔧 20 min — Vacuum house @home
  18:20-19:00  (40 min remaining)

Unscheduled:
  ⚠️ [t-055] 🎨 Creative 🔧 60 min @personal — no 🎨 Creative block today
     Suggestion: Add an evening 🎨 Creative block, or move to Saturday

📊 Summary:
  Scheduled: 6 tasks (215 min / 480 min available = 45%)
  Unscheduled: 1 task (no matching block)
  Calendar events: 3 (carving 90 min from blocks)
  Remaining capacity: 265 min across blocks
```

## Step 5: Confirm & Adjust

Ask: "Does this schedule work? Adjustments?"

User can:
- **Swap** task positions: "move t-032 to the afternoon"
- **Reassign** energy: "put t-055 in the admin block instead" (override energy match)
- **Add block**: "add a creative block from 20:00-21:00" (→ adds override to ENERGY-CALENDAR.md)
- **Remove task**: "drop t-035 for today, keep in ready"
- **Confirm**: "looks good"

## Step 6: Apply Schedule

1. Update tasks in today's daily file with scheduled times (add `Scheduled: HH:MM-HH:MM` metadata)
2. If user added energy block overrides, update `ENERGY-CALENDAR.md`
3. If Google Calendar MCP available, optionally create focus time blocks:
   - Ask: "Create calendar blocks for focus time? (helps others see you're busy)"
   - If yes, create events for each scheduled task block

## Step 7: Week View (if `/allocate-time week`)

Show capacity overview for the upcoming week:

```
📅 Week Capacity — March 19-25, 2026

          🧠 Deep    🤝 Social  ⚡ Admin   🎨 Creative  Total
Mon       3.0h @w    1.5h @w    3.5h       1.0h @p      9.0h
Tue       3.0h @w    1.5h @w    3.5h                    8.0h
Wed       5.0h @w               3.0h       1.0h @p      9.0h
*Thu      3.0h @w    1.5h @w    3.5h                    8.0h ← today
Fri       3.0h @w    2.0h @w    2.0h                    7.0h
Sat       1.0h @p               4.0h       2.0h @p      7.0h
Sun       (rest)                                         0.0h

Week:    21.0h       6.5h      19.5h       4.0h        48.0h

Tasks needing scheduling (not yet in-progress):
- [t-050] 🧠 Deep @work 480 min — Security audit (due Mar 25)
  Fits: Mon-Fri mornings, ~2.7 days of deep work
- [t-055] 🎨 Creative @personal 60 min — Blog post
  Fits: Mon/Wed/Sat evenings
```

## Git Commit

If `git.autoCommit` enabled:
```
Allocate time: scheduled {{N}} tasks for {{date}}

- {{task count}} tasks across {{block count}} energy blocks
- Total scheduled: {{time}} of {{available}} available

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Philosophy:** Make time real. Matching tasks to energy blocks makes plans concrete. The schedule is a suggestion — life happens, and you can always adjust. But starting with a plan grounded in your actual capacity beats winging it.
