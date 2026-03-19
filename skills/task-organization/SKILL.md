---
name: task-organization
description: This skill should be used when the user shares messy task thoughts, brain dumps, unstructured lists, todos, work items, or errands that need organizing into the OStaaT format. It applies when the user mentions "I need to", "I have to", "I should", pastes a list, describes multiple things to get done, or wants to use the /dump command. It should not be used for general codebase questions, single simple requests, or conversations that do not involve task capture.
---

# Task Organization Skill

Organize messy task thoughts into the OStaaT format and write them to the persistent task store. When a user mentions tasks or things they need to do, proactively structure them.

## OStaaT v4.0 Task Format

```markdown
- [t-NNN] 🔴 High ⏰ Now 🔧 45 min 📅 2026-03-05 #project-tag
  Task description
  Dependencies: t-NNN (if any)
  Context: (if any)
  Created: YYYY-MM-DD
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

### Energy Labels (if configured in ostaat.json)
- User-defined dimensions, e.g., `🧠 Deep`, `⚡ Admin`, `🎨 Creative`, `🤝 Social`
- Optional on tasks — stored as metadata

### Tags
- `#project-tag` — links to a project in PROJECTS.md
- `@area-tag` or `@area/sub-area` — links to an area in AREAS.md
- A task gets ONE tag type, never both

### Task IDs
- Every task gets a unique `[t-NNN]` ID from `tasks.nextId` in ostaat.json
- IDs are never recycled
- Reference other tasks by ID in dependencies: `Dependencies: t-017, t-031`

## How to Organize

1. **Extract tasks** from the user's input — don't invent or add anything not mentioned
2. **Identify obvious dependencies** between tasks
3. **Draft the list** using the format above (without IDs — those are assigned on write)
4. **Show the draft** and ask: "Does this capture everything?"
5. **Clarify** one question at a time for anything unclear (priority, timing, estimates)
6. **Offer to link** to projects or areas if they exist
7. **Assign IDs and write to task store** (`tasks/ready.md` or `tasks/inbox.md`)
8. **Update INDEX.md** with each new task
9. **Optionally add to today's focus** if daily file exists

## Hard Rules

- **Never invent tasks** — only organize what the user explicitly mentions
- **Resistance is opt-in ONLY** — never infer emotional states or hesitation
- **Subtasks are opt-in ONLY** — never break down tasks without permission
- **One question at a time** — don't overwhelm with clarification questions
- **Don't assume priorities** — ask if unclear, default to 🟡 Medium
- **Task store is authoritative** — always write to the store, optionally sync to daily file
- **Suggest /dump** if the user seems to have a lot to capture — it has a more structured flow

## Examples

### Example 1: Casual mention
**User says:** "I need to finish the quarterly report, call the accountant about taxes, and update my LinkedIn"

**Output (draft — IDs assigned on write):**
```markdown
- 🟡 Medium ⏭️ Next 🔧 120 min
  Finish quarterly report

- 🟡 Medium ⏭️ Next 🔧 20 min
  Call accountant about taxes

- 🟢 Low 📅 Later 🔧 30 min
  Update LinkedIn profile
```
Then ask about priorities, timing, and project/area linking. On confirmation, assign IDs and write to task store.

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

### Example 3: Brain dump with dependencies
**User says:** "Tomorrow I have that presentation — need to finish slides (3hrs), practice it (1hr), and make sure the demo works (30min). Can't practice until slides are done."

**Output:**
```markdown
- 🔴 High ⏰ Now 🔧 180 min 📅 2026-03-20
  Finish presentation slides

- 🔴 High ⏰ Now 🔧 60 min 📅 2026-03-20
  Practice presentation
  Dependencies: Finish presentation slides

- 🟡 Medium ⏰ Now 🔧 30 min 📅 2026-03-20
  Verify demo environment works
```
After IDs assigned: `Dependencies: t-042` (referencing the slides task).

## Integration

- Read `tasks/INDEX.md` — check for existing similar tasks before creating duplicates
- Read `YYYY-MM-DD-todo.md` — if today's file exists, offer to add tasks to focus
- Read `PROJECTS.md` and `AREAS.md` — offer to tag tasks
- Write to `tasks/ready.md` or `tasks/inbox.md` — task store is the destination
- Update `tasks/INDEX.md` — maintain the lookup table
- Update `ostaat.json` — increment `tasks.nextId`
- Respect `ostaat.json` defaults for priority and timing
- If `git.autoCommit` is enabled, commit after adding tasks
