# Energy Calendar & Constraint-Based Scheduling — Design Specification

**Version:** 4.0.0 (Phase 2)
**Status:** Implementing

---

## Overview

The energy calendar adds time-aware, capacity-based planning to OStaaT. It answers three questions that the task store alone cannot:

1. **When can I do this type of work?** — Energy blocks define time slots for different work types
2. **Do I have enough capacity before the deadline?** — The constraint engine works backward from due dates
3. **What should I work on right now?** — Matching available energy to task requirements

### Key concepts

- **Energy dimensions** are user-defined work categories (🧠 Deep, ⚡ Admin, 🎨 Creative, 🤝 Social)
- **Energy blocks** are time slots with an energy type and area scope (e.g., 09:00-12:00 🧠 Deep @work)
- **Area scoping** means a `@work` block can serve `@work`, `@work/engineering`, or `@work/management` tasks
- **Constraint detection** flags infeasibility: "Not enough 🧠 Deep @work time before March 31"

---

## Energy Calendar File

**Location:** `{workspace}/{dataDir}/ENERGY-CALENDAR.md`

```markdown
# Energy Calendar

## Weekly Defaults

### Monday
- 09:00-12:00 🧠 Deep @work
- 12:00-13:00 (break)
- 13:00-14:30 🤝 Social @work
- 14:30-17:00 ⚡ Admin @work
- 18:00-19:00 ⚡ Admin @home
- 20:00-21:00 🎨 Creative @personal

### Tuesday
- 09:00-12:00 🧠 Deep @work
- 12:00-13:00 (break)
- 13:00-14:30 🤝 Social @work
- 14:30-17:00 ⚡ Admin @work
- 18:00-19:00 ⚡ Admin @home

### Wednesday
- 09:00-12:00 🧠 Deep @work
- 12:00-13:00 (break)
- 13:00-15:00 🧠 Deep @work
- 15:00-17:00 ⚡ Admin @work
- 18:00-19:00 ⚡ Admin @home
- 20:00-21:00 🎨 Creative @personal

### Thursday
- 09:00-12:00 🧠 Deep @work
- 12:00-13:00 (break)
- 13:00-14:30 🤝 Social @work
- 14:30-17:00 ⚡ Admin @work
- 18:00-19:00 ⚡ Admin @home

### Friday
- 09:00-12:00 🧠 Deep @work
- 12:00-13:00 (break)
- 13:00-15:00 🤝 Social @work
- 15:00-17:00 ⚡ Admin @work

### Saturday
- 09:00-11:00 ⚡ Admin @home/maintenance
- 11:00-12:00 🧠 Deep @personal
- 14:00-16:00 🎨 Creative @personal

### Sunday
- (rest day)

## Overrides

- 2026-03-20: 09:00-17:00 🤝 Social @work (all-day offsite)
- 2026-03-21: (day off)
- 2026-03-25: +19:00-21:00 🧠 Deep @work (evening catchup)
- 2026-04-01 to 2026-04-05: (vacation)
```

### Block format

```
- HH:MM-HH:MM EnergyEmoji EnergyLabel @area[/sub-area] [(optional note)]
```

| Component | Required | Example |
|-----------|----------|---------|
| Time range | Yes | `09:00-12:00` |
| Energy emoji | Yes | `🧠` |
| Energy label | Yes | `Deep` |
| Area scope | Yes | `@work`, `@home/garden` |
| Note | No | `(team standup)` |

### Special entries

| Format | Meaning |
|--------|---------|
| `(break)` | Explicitly blocked — no work |
| `(rest day)` | Entire day off |
| `(day off)` | Override: entire day off |
| `(vacation)` | Multi-day override: all days off |
| `+HH:MM-HH:MM ...` | Override: add a block to the day's defaults |
| `YYYY-MM-DD to YYYY-MM-DD: ...` | Date range override |

### Override rules

- A date-specific override **replaces** the weekly default for that day entirely
- Use `+` prefix to **add** a block to the existing defaults instead of replacing
- Date range overrides apply to every day in the range
- Overrides are sorted by date

---

## Area Scoping

Energy blocks are scoped to areas. The area hierarchy determines which tasks can use which blocks.

### Matching rules

A task can use an energy block if:
1. The block's energy type matches the task's energy label
2. The block's area scope is **equal to or an ancestor of** the task's area tag

| Task tag | Can use blocks scoped to |
|----------|-------------------------|
| `@work/engineering` | `@work/engineering`, `@work` |
| `@work` | `@work` |
| `@home/garden` | `@home/garden`, `@home` |
| `@home` | `@home` |
| (no tag) | Any block (unscoped tasks are flexible) |
| `#project-tag` | Blocks matching the project's area (from PROJECTS.md `**Area:**` field), or any block if project has no area |

### Example

Given these blocks:
```
09:00-12:00 🧠 Deep @work
14:00-16:00 🧠 Deep @personal
```

- Task `[t-017] 🧠 Deep @work/engineering` → can use 09:00-12:00 (parent match)
- Task `[t-022] 🧠 Deep @personal` → can use 14:00-16:00 (exact match)
- Task `[t-030] 🧠 Deep #api-redesign` where project area is `@work` → can use 09:00-12:00
- Task `[t-040] 🧠 Deep` (no tag) → can use either block

---

## Constraint Engine

The constraint engine answers: "Given this task's requirements and deadline, when must I start?"

### Forward capacity calculation

For a given date range, calculate total available minutes per energy type per area:

```
Available 🧠 Deep @work (March 19-31):
  Mon-Fri: 3hrs/day × 9 weekdays = 27 hours = 1620 min
  Minus override Mar 20 (offsite, no deep blocks): -3 hrs
  Minus override Mar 21 (day off): -3 hrs
  = 21 hours = 1260 min available
```

### Backward planning from deadline

Given a task with a due date:
1. Sum total work needed (🔧 active time, including subtasks)
2. Identify matching energy blocks working **backward** from the due date
3. Subtract available block time until the work is covered
4. The earliest date where work could start = **latest viable start date**

```
Task: [t-030] Monthly financial close
  Due: 2026-03-31
  Total 🔧: 95 min (⚡ Admin)
  Area: @work (via #monthly-close project)

  Available ⚡ Admin @work blocks:
    Mar 31 (Mon): 14:30-17:00 = 150 min → covers 95 min

  Latest viable start: March 31 (enough capacity on deadline day)
```

### Infeasibility detection

If available capacity < required time before the deadline:

```
⚠️ Infeasibility detected for [t-050]:
  Task: Complete security audit
  Due: 2026-03-25
  Needs: 🧠 Deep @work — 480 min (8 hours)
  Available 🧠 Deep @work (now through Mar 25): 360 min (6 hours)

  Shortfall: 120 min (2 hours)

  Options:
  - Extend deadline
  - Add 🧠 Deep @work blocks (override)
  - Reduce scope / break into phases
  - Reassign to different energy type
```

### Dependency-aware planning

When tasks have dependencies, the constraint engine considers ordering:

```
t-031 (Pull bank statements) → must finish before t-032
t-032 (Reconcile accounts) → must finish before t-033, t-034
t-033 (Generate reports) + t-034 (Submit) → can run in parallel after t-032
```

The engine schedules dependent tasks in topological order, each consuming from the earliest available matching block after its dependencies complete.

---

## Integration with Commands

### `/start-day` — Energy-aware focus selection

When `energy.enabled` is true:

1. **Calculate today's energy budget** from ENERGY-CALENDAR.md:
   ```
   📊 Today's Energy Budget (Thursday):
   - 🧠 Deep @work: 3.0 hrs (09:00-12:00)
   - 🤝 Social @work: 1.5 hrs (13:00-14:30)
   - ⚡ Admin @work: 2.5 hrs (14:30-17:00)
   - ⚡ Admin @home: 1.0 hr (18:00-19:00)
   Total: 8.0 hrs
   ```

2. **Show energy fit** during focus selection:
   ```
   ✅ [t-017] 🧠 Deep 🔧 45 min @work/engineering — fits in Deep @work
   ✅ [t-048] ⚡ Admin 🔧 20 min @home — fits in Admin @home
   ⚠️ [t-055] 🎨 Creative 🔧 60 min @personal — no Creative block today
   ```

3. **Capacity check** after selection:
   ```
   📊 Energy Allocation:
   🧠 Deep @work: 2.5 / 3.0 hrs used (45 min remaining)
   🤝 Social @work: 0.0 / 1.5 hrs used
   ⚡ Admin @work: 1.5 / 2.5 hrs used (60 min remaining)
   ⚡ Admin @home: 0.3 / 1.0 hr used (40 min remaining)

   ⚠️ No 🎨 Creative block today — [t-055] won't fit unless you add one
   ```

4. **Infeasibility warnings** for tasks with upcoming deadlines:
   ```
   ⚠️ Deadline risk:
   - [t-050] Security audit — due Mar 25, needs 8 hrs 🧠 Deep @work, only 6 hrs available
   ```

### `/allocate-time` — Energy-aware time blocking

Enhanced to use energy blocks:

1. Read today's energy blocks from ENERGY-CALENDAR.md
2. If Google Calendar MCP available, overlay calendar events (meetings reduce available block time)
3. Match tasks to blocks by energy type + area scope
4. Propose a schedule:
   ```
   📅 Suggested Schedule (Thursday):

   09:00-09:45  [t-017] 🧠 Deep — API refactoring @work/engineering
   09:45-10:45  [t-050] 🧠 Deep — Security audit @work
   10:45-12:00  [t-052] 🧠 Deep — Design review @work

   12:00-13:00  (break)

   13:00-14:00  [t-060] 🤝 Social — Team standup + 1:1 @work
   14:00-14:30  (calendar: meeting with client)

   14:30-15:30  [t-048] ⚡ Admin — Update docs @work
   15:30-16:00  [t-031] ⚡ Admin — Pull bank statements @work

   18:00-18:20  [t-049] ⚡ Admin — Vacuum house @home
   18:20-19:00  ⚡ Admin @home — 40 min available

   Unscheduled (no matching block today):
   - [t-055] 🎨 Creative 🔧 60 min @personal
   ```

### `/review-day` — Energy usage report

At end of day, show energy utilization:
```
📊 Energy Usage Today:
🧠 Deep @work: 2.5 / 3.0 hrs (83%)
🤝 Social @work: 1.0 / 1.5 hrs (67%)
⚡ Admin @work: 2.0 / 2.5 hrs (80%)
⚡ Admin @home: 0.5 / 1.0 hr (50%)

Overall: 6.0 / 8.0 hrs utilized (75%)
```

### `/energy-calendar` — Manage the calendar

New command for viewing and editing the energy calendar.

---

## Configuration

When energy is fully enabled:

```json
{
  "energy": {
    "enabled": true,
    "dimensions": [
      { "label": "Deep", "emoji": "🧠", "description": "Deep focus, complex problem solving" },
      { "label": "Admin", "emoji": "⚡", "description": "Mechanical, administrative tasks" },
      { "label": "Creative", "emoji": "🎨", "description": "Design, writing, ideation" },
      { "label": "Social", "emoji": "🤝", "description": "Meetings, calls, collaboration" }
    ],
    "calendarFile": "ENERGY-CALENDAR.md",
    "showBudgetAtStartDay": true,
    "showUsageAtReviewDay": true,
    "warnOnNoMatchingBlock": true,
    "constraintCheckOnStartDay": true
  }
}
```

### Enforcement levels

| Setting | Behavior |
|---------|----------|
| `energy.enabled: false` | Energy labels stored on tasks but not enforced. No calendar. (Phase 1 default) |
| `energy.enabled: true` | Full energy calendar. Budget display, capacity checks, infeasibility warnings. |
| `warnOnNoMatchingBlock: true` | Warn when a task has no matching energy block today |
| `constraintCheckOnStartDay: true` | Run backward planning at `/start-day` for tasks with due dates |

---

## Weekly/Monthly Review Integration

### `/review-areas` and `/review-projects` — Energy adjustments

During periodic reviews, suggest energy calendar adjustments:

```
📊 Energy Capacity Review (this week):

Usage patterns:
- 🧠 Deep @work: 85% utilized (near capacity)
- 🤝 Social @work: 45% utilized (underused)
- ⚡ Admin @home: 30% utilized (overallocated?)
- 🎨 Creative @personal: 90% utilized (near capacity)

Suggestions:
- Consider converting some 🤝 Social @work → 🧠 Deep @work
- ⚡ Admin @home blocks could be shorter
- 🎨 Creative @personal might need more time

Adjust the energy calendar? (opens /energy-calendar)
```

---

## Philosophy

The energy calendar is an **awareness tool**, not a straightjacket:

- **It warns, not blocks.** You can always override and work on a 🧠 Deep task during an ⚡ Admin block.
- **It surfaces constraints early.** Better to know on March 19 that you can't finish by March 25 than to discover it on March 24.
- **It respects rhythms.** Different people have different energy patterns. The calendar is fully user-defined.
- **It's optional.** Everything works without it. Energy labels are metadata until you enable the calendar.
