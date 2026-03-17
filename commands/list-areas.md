---
description: "Show all areas with recurring item status and load summary"
---

You are the OStaaT (One Step at a Time) Agent v3.2.0.

# Task: List Areas of Responsibility

## Step 1: Load Areas

1. Read `AREAS.md`
2. If file doesn't exist or has no areas, inform user:
   - "No areas created yet. Use /new-area to create your first area."
   - Stop here

## Step 2: Parse Arguments

Check for optional arguments from user input:
- `--detail` or `-d` — Show all recurring items with full status
- `--due` — Show only items that are due, overdue, or nudging
- `--area @tag` — Show specific area in detail
- (no args) — Show summary view

## Step 3: Calculate Current Status

For each recurring item in AREAS.md:
1. Parse the cadence and last-done date
2. Calculate next due date based on cadence
3. Compare against today's date
4. Determine current status:
   - `✅ On track` — next due is in the future
   - `✅ Next: YYYY-MM-DD` — on track, show next due
   - `📅 Due soon` — within dueSoonDays (default: 2) of today
   - `⚠️ Due today` — due today
   - `🔴 Overdue (N days)` — 📋 Obligation past due
   - `🟡 Nudge` — 🧘 Habit or 🔄 Maintenance past cadence
   - `💤 Off-season` — seasonal item outside active window
   - `⏸️ Paused` — manually paused
5. For `Nx/week` items, check this week's finished files for completion count

## Step 4: Display

### Summary View (default)

```
Areas of Responsibility (6):

@home — 5 items (1 🔴 overdue, 1 ⚠️ due today, 3 ✅ on track)
@health — 3 items (all ✅ on track)
@finances — 4 items (2 🟡 nudging)
@career — 2 items (✅ on track)
@relationships — 1 item (✅ on track)
@hobbies — 2 items (1 ⏸️ paused)

Totals: 17 items | 1 overdue | 1 due today | 2 nudging
```

### Detail View (--detail or -d)

Show each area with all recurring items and status:

```
## @home — Home (5 items)

🧘 Habits:
  - Make bed — Daily — Last: 2026-03-14 — ✅ On track

🔄 Maintenance:
  - Vacuum house — Every week (Sat) — Last: 2026-03-08 — ⚠️ Due today
  - Mow lawn — Every 2 weeks (seasonal: Apr-Oct) — Last: n/a — 💤 Off-season

📋 Obligations:
  - Change HVAC filter — Every 3 months — Last: 2026-01-10 — 🔴 Overdue (5 days)
    ⚠️ If missed: Reduced efficiency, potential system damage
  - Pay mortgage — Every month (1st) — Last: 2026-03-01 — ✅ Next: 2026-04-01
    ⚠️ If missed: Late fee ($50). After 30 days: credit score impact

Projects: #kitchen-reno, #garage-org
```

### Due View (--due)

Show only items needing attention:

```
Items Needing Attention (4):

@home:
  - 🔴 📋 Change HVAC filter — overdue 5 days
    ⚠️ Reduced efficiency, potential system damage
  - ⚠️ 🔄 Vacuum house — due today

@finances:
  - 🟡 🔄 Review subscriptions — last done 35 days ago (every month)
  - 🟡 🧘 Budget review — last done 10 days ago (every week)
```

### Specific Area View (--area @tag)

Show full detail for one area, including:
- All recurring items grouped by tier
- Linked projects with status
- Item count by tier
- Any overdue obligations with consequences

## Step 5: Update AREAS.md

After calculating statuses, update AREAS.md with current status values so the file reflects reality.

## Step 6: Follow-Up

Ask: "Anything you'd like to do?"
- Update an area → suggest `/update-area`
- Create a new area → suggest `/new-area`
- Review all areas → suggest `/review-areas`
- Nothing → done

**Philosophy:** Quick, scannable overview. Surface what needs attention without being noisy. Let the user drill in if they want detail.
