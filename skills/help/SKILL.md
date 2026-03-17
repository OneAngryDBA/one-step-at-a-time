---
name: help
description: This skill should be used when the user asks about how OStaaT works, what commands are available, how to use the plugin, asks "what can you do", "how does this work", "what commands are there", "help me with OStaaT", "what's the difference between slash commands and natural language", or needs orientation with the task management system. It should not be used when the user is actively working on tasks, doing brain dumps, or running specific commands.
---

# OStaaT Help Skill

Help users understand and navigate the OStaaT system. When someone asks about how things work, what's available, or how to get started, guide them clearly.

## When to Activate

- User asks what OStaaT can do or what commands exist
- User asks how something works (areas, projects, daily workflow)
- User seems lost or unsure what to do next
- User asks about the difference between slash commands and natural language
- User asks for help with a specific command

## How to Help

### General "what can I do?" questions

Provide a concise overview organized by workflow phase:

1. **Getting started**: `/setup` to initialize, `/new-area` and `/new-project` to set up structure
2. **Daily rhythm**: `/start-day` → work → `/dump` or `/add-task` → `/review-day`
3. **Management**: `/list-projects`, `/list-areas`, `/review-projects`, `/review-areas`
4. **Integrations**: `/allocate-time` for calendar, `/pull` for external sources

Keep it brief. Don't list every command — highlight the ones most relevant to what the user seems to need.

### "How does X work?" questions

Read the relevant documentation and explain:
- For daily workflow questions → reference `docs/USAGE.md` and `docs/SYSTEM-ARCHITECTURE.md`
- For area questions → reference `docs/AREAS-DESIGN.md`
- For project questions → reference `docs/PROJECTS-DESIGN.md`
- For setup questions → reference `docs/SETUP.md`

### "Slash commands vs natural language" questions

Explain the two interaction modes:
- **Slash commands** are deterministic, full-featured, and consistent. Best for structured workflows.
- **Natural language** triggers proactive skills that recognize intent and either help directly or route to the right command.
- **Task capture** works well with natural language — just describe what you need to do.
- **Structured workflows** (start/end of day, reviews) work best as slash commands.

### Specific command help

If the user asks about a specific command, suggest they run `/ostaat-help <command>` for full details, or provide a brief explanation yourself by reading the relevant command file from `commands/`.

## Suggest Next Steps

After helping, suggest one concrete next action based on context:
- New user? → "Try `/setup` to get started"
- No daily file? → "Run `/start-day` to set up today"
- Seems to have tasks in mind? → "You can just tell me what you need to do and I'll organize it, or run `/dump` for a structured capture"
- Exploring? → "Run `/ostaat-help` for the full command reference"

## Hard Rules

- **Don't overwhelm** — answer the specific question, don't dump everything
- **One suggestion at a time** — suggest the most relevant next step
- **Point to `/ostaat-help`** for comprehensive reference rather than reproducing it all
- **Read docs when needed** — don't guess at system behavior, check the source
