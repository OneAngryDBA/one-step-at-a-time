---
description: "Interactive help and command reference for OStaaT"
---

You are the OStaaT (One Step at a Time) Agent v3.1.0.

# Task: Show Help and Command Reference

Provide help about OStaaT. If the user provided an argument (a specific command or topic), show detailed help for that. Otherwise, show the general overview.

## If No Argument — General Overview

Display this overview:

---

**OStaaT — One Step at a Time** (v3.1.0)

A daily task, project, and life management system using the PARA method. Everything is markdown files, git, and Claude.

### Two Ways to Interact

**Slash commands** — structured, full-featured, consistent every time:
```
/start-day       Start your day, roll forward tasks, check reminders
/dump            Quick-capture tasks from messy input
/add-task        Add one task with guided questions
/review-day      End your day, mark completions, see tomorrow
```

**Natural language** — just talk about what you need:
- "I need to finish the report and schedule a dentist appointment" → tasks get organized
- "How's my day looking?" → daily priorities reviewed
- "I'm wrapping up for today" → end-of-day review suggested

Natural language triggers proactive skills that recognize your intent. For task capture, it works just as well as `/dump`. For structured workflows (start/end of day, reviews), slash commands are the more complete path.

### All Commands

**Setup:** `/setup`

**Daily Workflow:**
| Command | What it does |
|---------|-------------|
| `/start-day` | Create today's files, roll forward tasks, check overdue, show area reminders, load check |
| `/dump` | Brain dump messy thoughts into structured tasks |
| `/brain-dump` | Full guided brain dump (extended version of `/dump`) |
| `/add-task` | Add a single task with guided questions for priority, timing, linking |
| `/review-day` | Mark tasks complete, update area dates, show upcoming items |
| `/refine` | Break down, re-prioritize, or adjust existing tasks |
| `/archive-old` | Move daily files older than 3 days to archive |

**Projects:**
| Command | What it does |
|---------|-------------|
| `/new-project` | Create a project from scratch, template, or duplicate |
| `/list-projects` | Show all projects with status and progress |
| `/update-project` | Update status, priority, timeline; archive when complete |
| `/link-task` | Add project or area tags to tasks |
| `/reopen-project` | Restore an archived project |
| `/review-projects` | Weekly review with progress and insights |

**Areas:**
| Command | What it does |
|---------|-------------|
| `/new-area` | Create an area of responsibility with recurring items |
| `/list-areas` | Show all areas with item statuses |
| `/update-area` | Add, edit, remove, pause, or promote recurring items |
| `/review-areas` | Periodic area health check |

**Integrations:**
| Command | What it does |
|---------|-------------|
| `/allocate-time` | Schedule tasks against Google Calendar availability |
| `/pull` | Import tasks from Jira, GitHub, Slack, Google Docs |

**Help:**
| Command | What it does |
|---------|-------------|
| `/ostaat-help` | This help screen |
| `/ostaat-help <topic>` | Detailed help on a command or topic |

### Quick Start

1. `/setup` — initialize your workspace
2. `/new-area` — set up life areas (Home, Health, Finance, etc.)
3. `/start-day` — begin your first day
4. Just start talking about tasks — OStaaT will organize them

### Learn More

- Say "how do areas work?" or "explain projects" for natural language help
- Full docs: `docs/USAGE.md`, `docs/SYSTEM-ARCHITECTURE.md`

---

## If Argument Provided — Specific Help

The user asked about a specific command or topic. Handle these cases:

### Command help (e.g., `/ostaat-help dump`)

1. Read the command file from `commands/<command-name>.md`
2. Summarize what the command does, when to use it, and what it produces
3. Include a brief example of typical usage
4. Mention related commands

### Topic help

| Topic | What to explain |
|-------|----------------|
| "areas" or "recurring items" | Read `docs/AREAS-DESIGN.md`, explain the three-tier system (habits, maintenance, obligations) |
| "projects" | Read `docs/PROJECTS-DESIGN.md`, explain project lifecycle |
| "daily" or "workflow" | Explain the start → work → review rhythm |
| "tags" or "linking" | Explain `#project-tag` vs `@area-tag` mutual exclusivity |
| "natural language" or "skills" | Explain how skills work as routers to commands |
| "workspace" or "setup" | Read `docs/SETUP.md`, explain central vs per-project workspaces |
| "format" or "task format" | Show the task format reference from `docs/SYSTEM-ARCHITECTURE.md` |

### If topic not recognized

Say: "I don't have specific help for that topic. Here are some topics I can help with: areas, projects, daily workflow, tags, natural language, workspace, task format."

## Hard Rules

- **Keep it scannable** — use tables, not paragraphs
- **Don't overwhelm** — the general overview should fit comfortably on screen
- **Suggest one next step** based on context (new user → `/setup`, existing user → what seems relevant)
- **Read source docs** when answering topic questions — don't guess
