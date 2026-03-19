---
description: "Create a new area of responsibility (or sub-area) with recurring items"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Create New Area of Responsibility

**Mode: CO-PILOT**

## Step 1: Area Name & Hierarchy

Ask: "What area of your life is this?" (e.g., Home, Health, Finances, Career, Relationships, Hobbies)

Then ask: "Is this a top-level area, or a sub-area of an existing one?"

**If sub-area:**
1. Read `AREAS.md` and list existing top-level areas
2. Ask which parent area this belongs under
3. The tag will use path notation: `@parent/child` (e.g., `@work/engineering`)

**If top-level:**
- Proceed as a standalone area

## Step 2: Generate Area Tag

1. Convert area name to tag format:
   - Lowercase
   - Replace spaces with hyphens
   - Remove special characters
   - Prefix with `@`
   - For sub-areas, use path: `@parent/child`
   - Examples:
     - "Home" → `@home`
     - "Engineering" under Work → `@work/engineering`
     - "Professional Development" → `@professional-development`
     - "Garden" under Home → `@home/garden`

2. Show generated tag
3. Ask: "Use this tag or specify custom tag?"

## Step 3: Add Recurring Items

Ask: "Want to add recurring items now? (You can always add more later with /update-area)"

If yes, for each item ask **one question at a time**:

1. **What's the item?** (e.g., "Vacuum house", "Pay credit card", "Exercise")

2. **What type is it?**
   - 🧘 **Habit** — Daily routine, no pressure (brush teeth, meditate, exercise)
   - 🔄 **Maintenance** — Periodic chore, soft deadline (vacuum, oil change, clean gutters)
   - 📋 **Obligation** — Hard deadline with real consequences (pay bills, file taxes, renew license)

3. **How often?** Accept natural language, normalize to standard cadence format:
   - Daily → `Daily`
   - Specific days → `Every Mon, Wed, Fri`
   - X times per week → `4x/week`
   - Weekly → `Every week` or `Every week (Sat)` if specific day
   - Every N weeks → `Every 2 weeks`
   - Monthly → `Every month` or `Every month (1st)` if specific date
   - Every N months → `Every 3 months`
   - Yearly → `Every year` or `Every year (Apr 15)` if specific date
   - Seasonal → append `(seasonal: Apr-Oct)` to any cadence

4. **When did you last do this?** (date, "today", "yesterday", "never", or approximate like "about a month ago")
   - Normalize to `YYYY-MM-DD` or `n/a` if never

5. **If 📋 Obligation:** "What happens if you miss it?"
   - Ask for immediate consequence
   - Ask: "Does it get worse over time? If so, what happens and when?"
   - Format as: `⚠️ If missed: {{immediate}}. After {{time}}: {{escalation}}`

6. **Template?** "Does this have a reusable checklist? (e.g., a template in templates/tasks/)"
   - If yes, add `Template: template-name` line
   - Also ask: "How many days lead time before the due date?" → add `Lead: Nd`

7. Ask: "Add another item?"
   - If yes, loop back to question 1
   - If no, proceed to Step 4

## Step 4: Create Area in AREAS.md

1. Read `AREAS.md` (create if it doesn't exist)
2. If the file only contains the placeholder text "(No areas created yet...)", replace the entire file content

**For top-level area:**
```markdown
## {{Area Name}} @{{area-tag}}
**Items:** {{count}} | **Projects:** 0

**Recurring:**
- {{tier_emoji}} {{description}} — {{cadence}} — Last: {{last_done}} — {{calculated_status}}
  {{if obligation: ⚠️ If missed: consequence line}}
  {{if template: Template: template-name}}
  {{if lead: Lead: Nd}}

**Projects:** (none)

---
```

**For sub-area (nested under parent):**
```markdown
### {{Sub-Area Name}} @{{parent/child-tag}}
**Items:** {{count}}

**Recurring:**
- {{tier_emoji}} {{description}} — {{cadence}} — Last: {{last_done}} — {{calculated_status}}
```

The sub-area section is placed INSIDE the parent area's section, after the parent's recurring items and before the parent's Projects line. Update the parent to show `**Sub-areas:** N`.

**Updated parent format with sub-areas:**
```markdown
## Work @work
**Items:** 8 | **Projects:** 3 | **Sub-areas:** 2

**Recurring:**
- 📋 Weekly status report — Every week (Fri) — Last: 2026-03-14 — ✅ Next: 2026-03-21

### Engineering @work/engineering
**Items:** 3 | **Projects:** 2

**Recurring:**
- 🔄 Review PRs — Every day (weekdays) — Last: 2026-03-18 — ✅ On track
- 📋 Sprint retrospective — Every 2 weeks (Fri) — Last: 2026-03-07 — 📅 Due soon
  Template: sprint-retro

### Management @work/management
**Items:** 2 | **Projects:** 1

**Recurring:**
- 📋 1:1 meetings — Every week (Tue) — Last: 2026-03-18 — ✅ On track

**Projects:** #api-redesign, #docs-update

---
```

3. Calculate status for each item based on cadence and last-done date
4. Update item count in the header

## Step 5: Link Existing Projects (Optional)

1. If `PROJECTS.md` exists and has active projects:
   - Ask: "Do any existing projects belong to this area?"
   - Show active projects list
   - If user selects any:
     - Add `**Area:** @{{area-tag}}` to each selected project in PROJECTS.md
     - Update the area's **Projects:** line with the project tags

2. If no projects or user skips, proceed

## Step 6: Git Commit

If `git.autoCommit` enabled:
1. Stage changes: `git add AREAS.md` (and `PROJECTS.md` if modified)
2. Commit with message:
   ```
   New area: {{area_name}} (@{{area-tag}})

   - {{if sub-area: Sub-area of @parent}}
   - Items: {{count}} ({{habits}} habits, {{maintenance}} maintenance, {{obligations}} obligations)

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

## Step 7: Summary

Show the created area with:
- Area name and tag (including path for sub-areas)
- All recurring items with current status
- Any linked projects
- Next steps: "Use /update-area to add more items, or /link-task to tag tasks with @{{area-tag}}"

**Philosophy:** Make area creation easy and guided. Don't overwhelm — start with a few items and build over time. Never assume what items someone should have. Only add what they explicitly mention. Sub-areas add structure when needed, not by default.
