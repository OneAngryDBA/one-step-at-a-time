# One Step at a Time (OStaaP) — Setup Guide

## Installation

```bash
claude plugin marketplace add https://github.com/OneAngryDBA/one-step-at-a-time.git
claude plugin install one-step-at-a-time
```

## After Installation

1. **Restart Claude Code** to load all commands and skills

2. **Run `/start-day`** to create your first daily files

3. **Try `/dump`** to add some tasks:
   ```
   /dump
   I need to finish the report, call the dentist, and buy groceries
   ```

4. **Create an area** with `/new-area`:
   ```
   /new-area
   → Home
   → Add items: vacuum (maintenance, weekly), pay mortgage (obligation, monthly)
   ```

5. **Create a project** with `/new-project` if you have multi-step goals

## Configuration

The plugin creates a `todo-config.json` in your working directory. Key settings:

- **areas.enabled** — Enable area tracking (default: true)
- **areas.loadCheck** — Show load check when thresholds exceeded (default: true)
- **projects.enabled** — Enable project management (default: true)
- **git.autoCommit** — Auto-commit after commands (default: true)

See `assets/config-template.json` for all options.

## Daily Workflow

1. **Morning**: `/start-day` — creates files, rolls forward tasks, shows reminders
2. **During day**: `/dump` or `/add-task` — capture new tasks
3. **End of day**: `/review-day` — mark complete, see tomorrow's items

## Periodic Reviews

- **Weekly**: `/review-projects` — project health check
- **Monthly**: `/review-areas` — area and recurring item review

## Natural Language

The plugin includes 4 skills that respond to natural language:
- **task-organization** — organizes casual task mentions
- **daily-workflow** — guides your daily rhythm
- **project-management** — helps with project-scale work
- **area-management** — helps with chores, habits, obligations

You don't need to use slash commands for everything — just talk naturally about your tasks and the skills will help.

## File Structure

```
your-project/
├── AREAS.md                    # Your areas of responsibility
├── PROJECTS.md                 # Your active projects
├── YYYY-MM-DD-todo.md          # Today's tasks
├── YYYY-MM-DD-finished.md      # Today's completions
├── scheduled/                  # Future-dated tasks
├── projects-archive/           # Completed projects
├── templates/                  # Project & area templates
└── todo-config.json            # Configuration
```
