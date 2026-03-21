# OStaaT v4.0 Test Checklist

**Test workspace:** ~/ostaat-test
**Plugin source:** /Users/matt/Documents/code/github/one-step-at-a-time
**Branch:** feature/v4-persistent-task-store

Start with:
```bash
mkdir -p ~/ostaat-test && cd ~/ostaat-test
script -q ~/ostaat-test-session.txt
claude --plugin-dir /Users/matt/Documents/code/github/one-step-at-a-time
```

---

## 1. Setup

### 1.1 Fresh workspace initialization
- [ ] Run `/setup`
- [ ] Choose: dedicated mode
- [ ] Choose: set up energy dimensions (use defaults: 🧠 Deep, ⚡ Admin, 🎨 Creative, 🤝 Social)
- [ ] Choose: set up energy calendar (basic weekday schedule)
- [ ] Choose: set as central workspace

**Verify files created:**
- [ ] `ostaat.json` exists with version 4.0.0
- [ ] `ostaat.json` has `tasks`, `energy`, `templates`, `hyperfocus` sections
- [ ] `tasks/INDEX.md` exists with empty table
- [ ] `tasks/inbox.md` exists with header
- [ ] `tasks/ready.md` exists with header
- [ ] `tasks/in-progress.md` exists with header
- [ ] `tasks/waiting.md` exists with header
- [ ] `tasks/paused.md` exists with header
- [ ] `AREAS.md` exists
- [ ] `PROJECTS.md` exists
- [ ] `templates/tasks/` directory exists
- [ ] `ENERGY-CALENDAR.md` exists (if energy calendar was set up)
- [ ] `~/.config/ostaat.json` points to test workspace
- [ ] `.gitignore` includes `.ostaat.lock`
- [ ] Git repo initialized with initial commit

**Notes:**


---

## 2. Task Capture

### 2.1 Add a single task
- [ ] Run `/add-task`
- [ ] Task: "Schedule dentist appointment"
- [ ] Priority: 🟡 Medium
- [ ] Timing: ⏭️ Next
- [ ] Time: 🔧 20 min
- [ ] Energy: ⚡ Admin
- [ ] Link to: skip (no areas yet)

**Verify:**
- [ ] Task appears in `tasks/ready.md` as `[t-001]`
- [ ] `tasks/INDEX.md` has row: t-001 | ready | Schedule dentist | —
- [ ] `ostaat.json` shows `tasks.nextId: 2`

**Notes:**


### 2.2 Quick dump — multiple tasks
- [ ] Run `/dump`
- [ ] Input: "need to buy groceries (30 min), pay electric bill (10 min, urgent), research vacation spots (45 min)"

**Verify:**
- [ ] 3 tasks created with sequential IDs (t-002, t-003, t-004)
- [ ] Electric bill should be 🔴 High or ⏰ Now (user said "urgent")
- [ ] All in `tasks/ready.md` (or inbox if details sparse)
- [ ] INDEX.md has 4 rows total
- [ ] `ostaat.json` shows `tasks.nextId: 5`

**Notes:**


### 2.3 Brain dump with dependencies
- [ ] Run `/brain-dump`
- [ ] Input: "I have a presentation Friday. Need to finish slides (3 hrs), practice it (1 hr), make sure demo works (30 min). Can't practice until slides are done."

**Verify:**
- [ ] 3 tasks created with IDs
- [ ] Practice task has `Dependencies: t-00X` referencing slides task
- [ ] Due dates set appropriately

**Notes:**


---

## 3. Task Management

### 3.1 List tasks
- [ ] Run `/list-tasks`

**Verify:**
- [ ] Shows all tasks grouped by status
- [ ] Shows summary stats (total, time estimates)

**Notes:**


### 3.2 Update task status
- [ ] Run `/update-task t-001 waiting`
- [ ] Provide: Waiting-on: "Dentist office callback"
- [ ] Expected date: tomorrow

**Verify:**
- [ ] t-001 removed from `tasks/ready.md`
- [ ] t-001 appears in `tasks/waiting.md` with Waiting-on and Expected fields
- [ ] INDEX.md shows t-001 status = waiting

**Notes:**


### 3.3 Update task back to ready
- [ ] Run `/update-task t-001 ready`

**Verify:**
- [ ] t-001 back in `tasks/ready.md`
- [ ] INDEX.md shows t-001 status = ready

**Notes:**


---

## 4. Areas & Hierarchy

### 4.1 Create a top-level area
- [ ] Run `/new-area`
- [ ] Name: "Work"
- [ ] Top-level area
- [ ] Add recurring item: "Weekly status report" — 📋 Obligation — Every week (Fri) — Last: last Friday
- [ ] Add recurring item: "Check email" — 🧘 Habit — Daily — Last: today

**Verify:**
- [ ] `AREAS.md` has `## Work @work` section
- [ ] Items count: 2
- [ ] Status calculated correctly (status report should be ✅ On track or similar)

**Notes:**


### 4.2 Create a sub-area
- [ ] Run `/new-area`
- [ ] Name: "Engineering"
- [ ] Sub-area of: Work
- [ ] Add recurring item: "Review PRs" — 🔄 Maintenance — Every day (weekdays) — Last: yesterday

**Verify:**
- [ ] `AREAS.md` has `### Engineering @work/engineering` nested under Work
- [ ] Parent shows **Sub-areas:** 1
- [ ] Sub-area tag is `@work/engineering`

**Notes:**


### 4.3 Create another top-level area
- [ ] Run `/new-area`
- [ ] Name: "Home"
- [ ] Add: "Vacuum" — 🔄 Maintenance — Every week (Sat) — Last: last Saturday

**Verify:**
- [ ] `AREAS.md` has both Work and Home sections

**Notes:**


### 4.4 Link a task to an area
- [ ] Run `/link-task t-001`
- [ ] Link to: @home

**Verify:**
- [ ] t-001 in `tasks/ready.md` now has `@home` tag
- [ ] INDEX.md tag column updated

**Notes:**


---

## 5. Projects

### 5.1 Create a project
- [ ] Run `/new-project`
- [ ] From scratch
- [ ] Name: "API Redesign"
- [ ] Priority: 🔴 High
- [ ] Due: 2 weeks from now
- [ ] Area: @work/engineering

**Verify:**
- [ ] `PROJECTS.md` has the project with `#api-redesign` tag
- [ ] Area field shows `@work/engineering`

**Notes:**


### 5.2 Link tasks to project
- [ ] Run `/add-task`
- [ ] Task: "Design new API endpoints"
- [ ] Link to: #api-redesign

**Verify:**
- [ ] New task has `#api-redesign` tag in store and INDEX.md

**Notes:**


---

## 6. Subtasks & Refinement

### 6.1 Break down a task into subtasks
- [ ] Run `/refine t-003` (the electric bill task, or whichever)
- [ ] Choose: Break down
- [ ] Create 2-3 subtasks

**Verify:**
- [ ] Subtasks have their own IDs from the same sequence
- [ ] Parent shows progress: (0/N done)
- [ ] Parent 🔧 time = sum of subtask times
- [ ] Subtasks nested under parent in status file
- [ ] INDEX.md: only parent listed (subtasks not in index)
- [ ] `ostaat.json` nextId incremented for subtasks

**Notes:**


---

## 7. Daily Workflow

### 7.1 Start day — focus selection
- [ ] Run `/start-day`
- [ ] Observe: in-progress tasks shown (should be empty first time)
- [ ] Observe: ready tasks shown sorted by priority
- [ ] Observe: waiting tasks flagged if expected date passed
- [ ] Observe: inbox items shown for triage (if any)
- [ ] Observe: area reminders shown
- [ ] If energy enabled: observe energy budget display
- [ ] Select 3-4 tasks for focus

**Verify:**
- [ ] Selected tasks moved to `tasks/in-progress.md`
- [ ] Today added to `Focus-dates` on each selected task
- [ ] INDEX.md statuses updated to `in-progress`
- [ ] Daily file `YYYY-MM-DD-todo.md` created with selected tasks
- [ ] Daily file `YYYY-MM-DD-finished.md` created (empty)
- [ ] Tasks in daily file have full details including IDs

**Notes:**


### 7.2 Add a task mid-day
- [ ] Run `/add-task`
- [ ] Task: "Urgent: server is down, fix it"
- [ ] Priority: 🔴 High, Timing: ⏰ Now
- [ ] Choose: add to today's focus

**Verify:**
- [ ] Task in `tasks/in-progress.md` with new ID
- [ ] Task also in today's daily file
- [ ] INDEX.md updated

**Notes:**


### 7.3 Review day — mark completions
- [ ] Run `/review-day`
- [ ] Mark 2-3 tasks as complete (by ID)
- [ ] For remaining tasks: keep in progress

**Verify:**
- [ ] Completed tasks removed from `tasks/in-progress.md`
- [ ] INDEX.md shows completed tasks as `done`
- [ ] Completed tasks written to `YYYY-MM-DD-finished.md`
- [ ] If any completed task had @area-tag + 🔄 Recurring: AREAS.md Last date updated
- [ ] If subtask completed: parent progress count updated
- [ ] If energy enabled: energy usage report shown
- [ ] Remaining tasks still in `tasks/in-progress.md`

**Notes:**


---

## 8. Templates

### 8.1 Create and instantiate a template
- [ ] Check that `templates/tasks/example-checklist.md` exists
- [ ] Run `/new-from-template example-checklist`
- [ ] Provide variables: month-name = "March", year = "2026"

**Verify:**
- [ ] 4 tasks created with sequential IDs
- [ ] Variables replaced in task descriptions
- [ ] Dependencies mapped to new task IDs
- [ ] All tasks in `tasks/ready.md`
- [ ] INDEX.md has 4 new rows

**Notes:**


---

## 9. Energy Calendar

### 9.1 View energy calendar
- [ ] Run `/energy-calendar today`

**Verify:**
- [ ] Shows today's blocks with times, energy types, area scopes
- [ ] Shows total hours

**Notes:**


### 9.2 Add an override
- [ ] Run `/energy-calendar override` for tomorrow
- [ ] Add: day off (or modified schedule)

**Verify:**
- [ ] `ENERGY-CALENDAR.md` has override in Overrides section

**Notes:**


### 9.3 View capacity
- [ ] Run `/energy-calendar capacity` for next 7 days

**Verify:**
- [ ] Shows capacity breakdown by energy type and area
- [ ] Override day reflected in totals

**Notes:**


---

## 10. Focus Modes

### 10.1 Panic mode
- [ ] Run `/panic`

**Verify:**
- [ ] Damage report shows: overdue items, at-risk, blocked, current load
- [ ] Triage proposed: must / should / drop
- [ ] Can adjust buckets
- [ ] Survival plan generated
- [ ] Daily file regenerated with MUST items
- [ ] Deferred tasks moved to paused with reason

**Notes:**


### 10.2 Hyperfocus mode
- [ ] Run `/hyperfocus #api-redesign` (or a task ID)

**Verify:**
- [ ] Target assessed: work needed, dependencies, feasibility
- [ ] Blast radius shown: what will be deferred
- [ ] Can keep specific items alongside target
- [ ] Non-target tasks moved to paused with `Paused-reason: Hyperfocus...`
- [ ] Daily file regenerated with target tasks only
- [ ] `ostaat.json` has `hyperfocus.active: true` with target info

**Notes:**


### 10.3 Exit hyperfocus
- [ ] Run `/hyperfocus off`

**Verify:**
- [ ] Paused tasks restored to ready
- [ ] `Paused-reason: Hyperfocus...` removed
- [ ] `ostaat.json` has `hyperfocus.active: false`
- [ ] Shows what was waiting

**Notes:**


---

## 11. Allocate Time

### 11.1 Energy-aware scheduling
- [ ] Run `/allocate-time`

**Verify:**
- [ ] Tasks matched to energy blocks by type + area
- [ ] Schedule proposed with time slots
- [ ] Unscheduled tasks flagged (no matching block)
- [ ] Capacity summary shown

**Notes:**


---

## 12. List & Query

### 12.1 Filter by status
- [ ] Run `/list-tasks --status ready`
- [ ] Verify: only ready tasks shown

### 12.2 Filter by tag
- [ ] Run `/list-tasks --tag #api-redesign`
- [ ] Verify: only project tasks shown

### 12.3 Filter by area
- [ ] Run `/list-tasks --tag @work`
- [ ] Verify: work-area tasks shown

**Notes:**


---

## 13. Multi-Day Persistence

### 13.1 Next day start
- [ ] Exit and restart Claude session
- [ ] Run `/start-day`

**Verify:**
- [ ] Yesterday's in-progress tasks shown (pre-selected)
- [ ] No "📌 Carried over from" notes — uses Focus-dates instead
- [ ] Ready backlog available for selection
- [ ] New daily file generated

**Notes:**


---

## 14. Upgrade Path

### 14.1 Test /upgrade detection
- [ ] Run `/upgrade`

**Verify:**
- [ ] Detects current version as 4.0.0
- [ ] Reports: "Your workspace is already on the latest version"

**Notes:**


---

## Summary

| Test | Pass | Fail | Notes |
|------|------|------|-------|
| 1. Setup | | | |
| 2. Task Capture | | | |
| 3. Task Management | | | |
| 4. Areas & Hierarchy | | | |
| 5. Projects | | | |
| 6. Subtasks & Refinement | | | |
| 7. Daily Workflow | | | |
| 8. Templates | | | |
| 9. Energy Calendar | | | |
| 10. Focus Modes | | | |
| 11. Allocate Time | | | |
| 12. List & Query | | | |
| 13. Multi-Day Persistence | | | |
| 14. Upgrade Path | | | |

**Transcript location:** `~/.claude/projects/` (auto-saved)
**Terminal recording:** `~/ostaat-test-session.txt` (if using `script`)
