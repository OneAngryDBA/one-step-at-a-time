---
name: task-organization
description: This skill should be used when the user shares messy task thoughts, brain dumps, unstructured lists, todos, work items, or errands that need organizing into the Action Organizer format. It applies when the user mentions "I need to", "I have to", "I should", pastes a list, describes multiple things to get done, or wants to use the /dump command. It should not be used for general codebase questions, single simple requests, or conversations that do not involve task capture.
---

# Task Organization Skill

Organize messy task thoughts into the Action Organizer format. When a user mentions tasks or things they need to do, proactively structure them.

## Action Organizer Task Format

```markdown
- 🔴 High ⏰ Now 🔧 45 min 📅 2026-03-05 #project-tag
  Task description
  Dependencies: (if any)
  Context: (if any)
```

### Priority
- 🔴 High — urgent or critical
- 🟡 Medium — important but not urgent
- 🟢 Low — nice to have, can wait

### Timing
- ⏰ Now — do today
- ⏭️ Next — do soon
- 📅 Later — can wait

### Time Estimates
- 🔧 active minutes (required) — time actively working
- 🕓 passive minutes (optional) — waiting time

### Tags
- `#project-tag` — links to a project in PROJECTS.md
- `@area-tag` — links to an area in AREAS.md
- A task gets ONE tag type, never both

## How to Organize

1. **Extract tasks** from the user's input — don't invent or add anything not mentioned
2. **Identify obvious dependencies** between tasks
3. **Draft the list** using the format above
4. **Show the draft** and ask: "Does this capture everything?"
5. **Clarify** one question at a time for anything unclear (priority, timing, estimates)
6. **Offer to link** to projects or areas if they exist
7. **Add to today's file** (`YYYY-MM-DD-todo.md`) when confirmed

## Hard Rules

- **Never invent tasks** — only organize what the user explicitly mentions
- **Resistance is opt-in ONLY** — never infer emotional states or hesitation
- **Subtasks are opt-in ONLY** — never break down tasks without permission
- **One question at a time** — don't overwhelm with clarification questions
- **Don't assume priorities** — ask if unclear, default to 🟡 Medium
- **Suggest /dump** if the user seems to have a lot to capture — it has a more structured flow

## Examples

### Example 1: Casual mention
**User says:** "I need to finish the quarterly report, call the accountant about taxes, and update my LinkedIn"

**Output:**
```markdown
- 🟡 Medium ⏭️ Next 🔧 120 min
  Finish quarterly report

- 🟡 Medium ⏭️ Next 🔧 20 min
  Call accountant about taxes

- 🟢 Low 📅 Later 🔧 30 min
  Update LinkedIn profile
```
Then ask about priorities, timing, and project/area linking.

### Example 2: Pasted list with some priorities
**User says:** "Here's what I've got: deploy API changes (urgent, 45min), review Sarah's PR (30min), update docs (low priority, 15min)"

**Output:**
```markdown
- 🔴 High ⏰ Now 🔧 45 min
  Deploy API changes

- 🟡 Medium ⏭️ Next 🔧 30 min
  Review Sarah's PR

- 🟢 Low 📅 Later 🔧 15 min
  Update docs
```
User provided some priorities — use them. Ask about the rest.

### Example 3: Brain dump with dependencies
**User says:** "Tomorrow I have that presentation — need to finish slides (3hrs), practice it (1hr), and make sure the demo works (30min). Can't practice until slides are done."

**Output:**
```markdown
- 🔴 High ⏰ Now 🔧 180 min 📅 2026-03-16
  Finish presentation slides

- 🔴 High ⏰ Now 🔧 60 min 📅 2026-03-16
  Practice presentation
  Dependencies: Finish presentation slides

- 🟡 Medium ⏰ Now 🔧 30 min 📅 2026-03-16
  Verify demo environment works
```
Dependency detected from user's input — captured, not invented.

## Integration

- Read `YYYY-MM-DD-todo.md` — if today's file exists, offer to add tasks
- Read `PROJECTS.md` and `AREAS.md` — offer to tag tasks
- Respect `todo-config.json` defaults for priority and timing
- If `git.autoCommit` is enabled, commit after adding tasks
