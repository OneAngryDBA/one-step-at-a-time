# OStaaT Roadmap

Planned enhancements and ideas for future development, organized by theme. This is a living document — items here are ideas under consideration, not commitments.

Want to suggest something? Open an issue on [GitHub](https://github.com/OneAngryDBA/one-step-at-a-time/issues).

---

## Task Dependencies

Currently, dependencies are a free-text `Dependencies:` line on tasks — a note for humans, not enforced by the system.

**Planned enhancements:**

- [ ] **Structured task-to-task linking** — Task B explicitly depends on Task A completing. When Task A is marked done, surface Task B as unblocked.
- [ ] **Blocked status** — Tasks with unresolved dependencies show a `🚫 Blocked` status and are excluded from prioritization until unblocked.
- [ ] **Failure/conditional branching** — If Task A fails or is cancelled, optionally trigger alternate tasks or flag dependents for review.
- [ ] **Wait-time dependencies** — "Start Task B two days after Task A completes" — time-delayed dependencies for things like curing, shipping, or approval waits.
- [ ] **Dependency visualization** — Show a task's dependency chain in `/review-day` or `/refine` so you can see what's blocked and what unblocks what.

**Design considerations:**
- Must stay markdown-friendly — no database, no IDs. Tag-based or name-based linking.
- Should not add friction to task capture — dependencies are optional and can be added later via `/refine` or `/link-task`.
- The system should surface blockers, not enforce them — the user always decides.

---

## Project Dependencies

Projects can already reference each other informally (e.g., `Feeds into: #pisa-trip`), but there's no structured system.

- [ ] **Project-to-project dependencies** — One project blocks another. `/review-projects` surfaces blocked projects and shows the dependency chain.
- [ ] **Gantt chart generation** — Auto-generate Mermaid Gantt charts from project milestones and timelines.
- [ ] **Cross-project task visibility** — When reviewing a project, see tasks from dependent projects that are blocking progress.

---

## Time & Scheduling

- [ ] **Smart rescheduling** — Suggest new dates based on current capacity and load thresholds when tasks are overdue or days are overloaded.
- [ ] **Time blocking** — Deeper Google Calendar integration to create focus time blocks for tasks, not just read availability.
- [ ] **Passive time concurrency** — When scheduling, suggest tasks that can run during another task's passive time (e.g., do email while waiting for a deploy).
- [ ] **Due date history** — Track how often tasks are rescheduled to identify patterns (chronic rescheduling may indicate task needs breakdown or deprioritization).

---

## Integrations

- [ ] **Calendar integration** — Show due dates on Google Calendar, sync both directions.
- [ ] **Jira/GitHub auto-import** — Pull assigned issues as tasks with project linking, keep status in sync.
- [ ] **Slack reminders** — Surface overdue obligations or upcoming deadlines as Slack reminders.
- [ ] **Team due dates** — Support tasks assigned to others with visibility in your daily file.

---

## Areas & Recurring Items

- [ ] **Recurring due dates on tasks** — Weekly tasks, monthly reviews — tasks that auto-regenerate on a schedule (distinct from area recurring items).
- [ ] **Streak tracking for habits** — "Exercise: 12-day streak" to encourage consistency without guilt.
- [ ] **Seasonal auto-pause/resume** — Maintenance items with active windows (e.g., "Apr-Oct") automatically pause and resume based on date.

---

## Daily Workflow

- [ ] **Notifications** — Remind day before a due date (requires integration with notification system).
- [ ] **Daily summary export** — Generate a shareable daily summary (what was done, what's planned) for standups or status updates.
- [ ] **Weekly digest** — Auto-generated end-of-week summary combining `/review-projects` and `/review-areas` data.

---

## Git & History

- [ ] **Remote push automation** — Optionally auto-push after commits for backup.
- [ ] **Daily branch creation** — One branch per day for cleaner history.
- [ ] **Commit-to-issue linking** — Link OStaaT commits to Jira/GitHub issues automatically.
- [ ] **Commit history visualization** — Show task completion trends over time from git history.

---

## Visualization & Reporting

- [ ] **Progress dashboards** — Mermaid-based charts showing project progress, area health, and task completion trends.
- [ ] **Gantt charts from milestones** — Auto-generate project timeline visualizations.
- [ ] **Load trend analysis** — "Your average daily load this week was 6.2 tasks / 4.1 hours" — patterns over time.
- [ ] **Budget/cost tracking** — Optional cost field on projects for tracking expenses.

---

## Plugin & UX

- [ ] **Task templates** — Reusable task templates for common workflows (e.g., "deploy checklist", "PR review steps").
- [ ] **Undo support** — "Undo last action" to recover from mistakes without git gymnastics.
- [ ] **Multi-day view** — See tasks across the next 3-5 days, not just today and tomorrow.
- [ ] **Archive search** — Search completed tasks across archived daily files.
