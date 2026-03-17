# Git Commit Workflow Design

## Overview

The OStaaP system automatically commits changes after major actions to maintain a history of your daily workflow and system improvements.

---

## Commit Strategy

### When Commits Happen

**Auto-commit triggers** (configurable in `todo-config.json`):
- After `/start-day` - Daily initialization
- After `/dump` or `/brain-dump` - Brain dump sessions
- After `/add-task` - Single task additions (batched if multiple in succession)
- After `/review-day` - End-of-day review
- After `/refine` - Task refinements
- After `/archive-old` - Archiving old files

**Threshold logic:**
- Minimum 1 file changed
- Or minimum 2 tasks added/modified
- "Large enough changes" = either threshold met

### What Gets Tracked

**Everything:**
- System files (`.claude/commands/`, `todo-config.json`, `ENHANCEMENTS.md`)
- Daily todo files (`YYYY-MM-DD-todo.md`)
- Finished files (`YYYY-MM-DD-finished.md`)
- Archive folder and all archived files
- Git workflow documentation

---

## Commit Message Template

**Format:**
```
{{action}}: {{summary}}

{{details}}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Template Variables:**
- `{{action}}` - The slash command that triggered the commit (e.g., "Start day", "Brain dump", "Review day")
- `{{summary}}` - One-line summary of what changed (e.g., "Added 5 tasks, rolled forward 2")
- `{{details}}` - Multi-line details:
  - Files changed
  - Tasks added/completed/modified
  - Any significant changes to system files

**Examples:**

```
Start day: Initialize 2026-02-25

- Created 2026-02-25-todo.md
- Created 2026-02-25-finished.md
- Rolled forward 2 incomplete tasks from 2026-02-24

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

```
Brain dump: Added 5 tasks for client work

- Added 5 new tasks to 2026-02-25-todo.md
- Tasks include: API integration, bug fixes, documentation updates
- Estimated total time: 🔧 180 min

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

```
Review day: Completed 3 tasks

- Moved 3 tasks to 2026-02-25-finished.md
- Total active time: 🔧 90 min
- 2 tasks remain for tomorrow

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Implementation Notes

### In Slash Commands

Each slash command that triggers commits should:
1. Complete its primary action
2. Check if `git.autoCommit` is enabled in config
3. Check if changes meet the threshold
4. Generate commit message using template
5. Stage relevant files (`git add`)
6. Create commit with generated message

### Manual Commits

Users can also manually commit anytime using standard git commands:
- `git add .`
- `git commit -m "your message"`

The system doesn't prevent manual commits - auto-commits just reduce friction.

---

## Configuration

In `todo-config.json`:

```json
"git": {
  "autoCommit": true,                    // Enable/disable auto-commits
  "commitOnActions": [                   // Which commands trigger commits
    "/start-day",
    "/dump",
    "/brain-dump",
    "/add-task",
    "/review-day",
    "/refine",
    "/archive-old"
  ],
  "commitThreshold": {
    "minChanges": 1,                     // Minimum files changed
    "minTasks": 2                        // OR minimum tasks added/modified
  },
  "messageTemplate": {
    "enabled": true,                     // Use template vs. simple messages
    "format": "..."                      // Template format (see above)
  }
}
```

---

## Future Enhancements

- Remote push automation (optional)
- Daily branch creation (one branch per day)
- Integration with project management tools (link commits to Jira/GitHub issues)
- Commit history visualization in daily files
