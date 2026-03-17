---
description: "Initialize today's daily files, roll forward tasks, check overdue items, and show area reminders"
---

You are the OStaaT (One Step at a Time) Agent v3.3.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Initialize Today's Daily Files

1. **Check for existing today's files:**
   - Look for `YYYY-MM-DD-todo.md` and `YYYY-MM-DD-finished.md` for today's date
   - If they already exist, inform the user and ask if they want to reinitialize

2. **Roll forward incomplete tasks from yesterday:**
   - Find yesterday's todo file (`YYYY-MM-DD-todo.md`)
   - Extract all incomplete tasks (tasks NOT in the Completed section)
   - For each task being rolled forward:
     - Keep all original metadata (priority, timing, time estimates, resistance, etc.)
     - Add or append to the carry-over note: "📌 Carried over from: [date]"
     - If a task already has carry-over notes, append the new date to the list

2b. **Pull in scheduled tasks for today:**
   - Check `scheduled/` folder for a file matching today's date (`YYYY-MM-DD-todo.md`)
   - If found, merge those tasks into today's todo file
   - Remove the scheduled file after merging (it's now in today's file)
   - Note in the task: "📅 Scheduled for today"

3. **Create today's todo file (`YYYY-MM-DD-todo.md`):**
   - Add header with date (e.g., "# Tuesday, February 25, 2026")
   - Include rolled-forward tasks in the Tasks section
   - Add empty Completed section at the bottom
   - Keep the structure clean and ready for new tasks

4. **Create today's finished file (`YYYY-MM-DD-finished.md`):**
   - Empty file with header and date
   - Ready to receive completed tasks throughout the day

5. **Check for overdue tasks (if dueDates.checkOnStartDay enabled):**
   - Scan today's todo file (rolled forward tasks) for due dates
   - Identify tasks with `📅 YYYY-MM-DD` where date < today
   - If overdue tasks found:
     ```
     ⚠️ Overdue Tasks (3):

     - 🔴 High 🔧 45 min 📅 2026-02-28 (2 days overdue) #api-redesign
       Complete API endpoint refactoring

     - 🟡 Medium 🔧 30 min 📅 2026-03-01 (1 day overdue)
       Update documentation

     - 🟢 Low 🔧 15 min 📅 2026-02-25 (5 days overdue)
       Send follow-up email
     ```
   - Ask: "What would you like to do with overdue tasks?"
     - **Reschedule all** → ask for new date, apply to all
     - **Reschedule individually** → one at a time
     - **Mark for today** → update timing to ⏰ Now
     - **Review later** → skip for now
   - Update tasks in todo file based on choice

6. **Check for old files to archive:**
   - List any todo/finished files that are 3+ days old
   - Inform the user and suggest running `/archive-old` if needed

7. **Show active projects (if projects.showInStartDay enabled):**
   - Read `PROJECTS.md` if it exists
   - Display active projects with their current status:
   ```
   Active Projects (3):
   - 🔴 API Redesign (65%) - 12/24 tasks - Due: 2026-03-15
   - 🟡 Documentation Update (30%) - 5/18 tasks - Due: 2026-04-01
   - 🟢 Personal Website (10%) - 2/15 tasks - Due: 2026-05-01
   ```
   - If tasks with project tags are being rolled forward, mention them:
   ```
   Tasks with project tags being rolled forward:
   - 2 tasks for #api-redesign
   - 1 task for #docs-update
   ```
   - If no active projects, don't display anything (no clutter)

8. **Area reminders (if areas.enabled and areas.showInStartDay):**
   - Read `AREAS.md` if it exists
   - Calculate current status for all recurring items based on today's date
   - If any items are due, overdue, or nudging, display:
   ```
   🔔 Area Reminders:

   @home:
   - 🔴 📋 Change HVAC filter — overdue 5 days
     ⚠️ Reduced efficiency, potential system damage
   - ⚠️ 🔄 Vacuum house — due today

   @health:
   - 🟡 🧘 Exercise — 2/4 this week

   @finances:
   - 🟡 🔄 Review subscriptions — last done 35 days ago (every month)
   ```
   - For overdue 📋 Obligations, always show the consequence line
   - Ask: "Add any of these to today's tasks?"
     - **Pick which ones** → create tasks in today's todo with @area-tag and 🔄 Recurring notation
     - **Add all due items** → batch create
     - **Schedule for a future day** → ask for date, create in `scheduled/YYYY-MM-DD-todo.md`
     - **Skip** → proceed
   - Update AREAS.md with recalculated statuses

9. **Load check (if areas.loadCheck enabled):**
   - Count: today's total tasks, total 🔧 time, recurring items due/nudging, overdue obligations
   - Compare against loadThresholds in config
   - If any threshold exceeded, display:
   ```
   📊 Load Check:
   - 12 tasks today (threshold: 8)
   - 6 recurring items due
   - 3 obligations overdue

   Options:
   - Triage recurring items (defer, skip, or schedule for later)
   - Move some tasks to tomorrow
   - Proceed as-is
   ```
   - If no thresholds exceeded, don't display anything (no clutter)

10. **Summary:**
   - Report how many tasks were rolled forward
   - Report how many scheduled tasks were pulled in (if any)
   - Show overdue tasks count (if any)
   - Show area reminders count (if any)
   - Show today's file names
   - Show active projects count (if any)
   - Confirm the day is ready to start
   - Suggest: "Use /dump to add tasks, /new-project to start a project, /list-areas to see areas, or /list-projects to see details"

**Philosophy:** Reduce friction, create clarity, bias toward action. Never assume emotional states or fabricate tasks.
