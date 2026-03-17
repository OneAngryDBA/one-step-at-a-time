# One Step at a Time (OStaaT)

A comprehensive daily task, project, and life management plugin for Claude Code using the PARA method. One step at a time.

## What It Does

- **Transform messy thoughts** into organized, actionable tasks
- **Track projects** with goals, timelines, and auto-calculated progress
- **Manage areas of life** (Home, Health, Finances) with recurring items
- **Three-tier recurring items**: Habits, Maintenance, Obligations with consequence tracking
- **Daily workflow** with automatic task rollover and gentle reminders
- **Load awareness** — data-driven, never assumes emotions
- **Central workspace** — manage tasks from any Claude Code project

## Install

```bash
claude plugin marketplace add https://github.com/OneAngryDBA/one-step-at-a-time.git
claude plugin install one-step-at-a-time
```

Then restart Claude Code and run `/setup` to initialize your workspace.

## How It Works

OStaaT installs globally and uses a **central workspace** — a single directory (and git repo) where all your tasks, projects, and areas live. You can run OStaaT commands from **any** Claude Code project and they all read/write to the same place.

```
~/.config/ostaat.json          → points to your central workspace
~/Documents/ostaat/             → your tasks, projects, areas (git repo, KEEP PRIVATE)
~/code/any-project/             → run /start-day, /dump, etc. from here — uses central workspace
~/code/client-project/
  └── .claude/one-step-at-a-time.local.md  → optional: override to use a local workspace
```

### Workspace Modes

- **Dedicated** — the repo exists for OStaaT. Data files at root.
- **Shared** — the repo has code too. OStaaT files live in `.ostaat/` subdirectory.

Run `/setup` to configure.

## Commands (20)

### Setup
| Command | Description |
|---------|-------------|
| `/setup` | Initialize OStaaT workspace (central or per-project) |

### Daily Workflow
| Command | Description |
|---------|-------------|
| `/start-day` | Initialize today, roll forward tasks, show reminders |
| `/dump` | Quick task capture |
| `/brain-dump` | Full guided brain dump |
| `/add-task` | Add a single task with guided questions |
| `/review-day` | Mark complete, review upcoming items |
| `/refine` | Refine existing tasks |
| `/archive-old` | Archive files older than 3 days |

### Projects
| Command | Description |
|---------|-------------|
| `/new-project` | Create a new project |
| `/list-projects` | Show all projects with status |
| `/update-project` | Update project details or archive |
| `/link-task` | Link tasks to projects or areas |
| `/reopen-project` | Restore archived project |
| `/review-projects` | Weekly project review |

### Areas
| Command | Description |
|---------|-------------|
| `/new-area` | Create a new area of responsibility |
| `/list-areas` | Show areas with recurring item status |
| `/update-area` | Manage recurring items, promote to project |
| `/review-areas` | Periodic area review |

### Integrations
| Command | Description |
|---------|-------------|
| `/allocate-time` | Schedule against Google Calendar |
| `/pull` | Pull from Jira, GitHub, Slack |

## Skills (5)

The plugin includes proactive skills that respond to natural language:
- **workspace-resolution** — resolves which workspace to use before any command
- **task-organization** — organizes casual task mentions
- **daily-workflow** — guides your daily rhythm
- **project-management** — helps with project-scale work
- **area-management** — helps with chores, habits, obligations

## Documentation

- [Setup Guide](docs/SETUP.md)
- [System Architecture](docs/SYSTEM-ARCHITECTURE.md)
- [Areas Design](docs/AREAS-DESIGN.md)
- [Projects Design](docs/PROJECTS-DESIGN.md)

## License

MIT
