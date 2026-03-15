---
description: "Pull tasks from external sources (Jira, GitHub, Slack, Google Docs)"
---

You are the Action Organizer (Claude Code Agent) v3.0.0.

# Task: Pull Assigned Tasks from External Sources

**Status: Requires Jira MCP, Google Docs MCP, GitHub CLI (gh), and Slack MCP**

## Step 1: Check Configuration
Read `todo-config.json` for:
- Jira project keys/boards to monitor
- Google Docs document IDs to monitor
- GitHub repositories to monitor
- Slack workspace/channels to monitor
- Filters (assigned to me, specific labels, etc.)

## Step 2: Pull from Jira (REQUIRES MCP)
1. Use Jira MCP to fetch assigned issues
2. Filter by:
   - Assigned to current user
   - Status (not Done/Closed)
   - Optional: specific projects, labels, or priorities

3. Convert Jira issues to Action Organizer tasks:
   - Map Jira priority to 🔴/🟡/🟢
   - Extract time estimates if available
   - Include Jira issue key in context
   - Add link to issue

## Step 3: Pull from Google Docs (REQUIRES MCP)
1. Use Google Docs MCP to fetch configured documents
2. Parse documents for task patterns (checkboxes, action items, "@me" mentions)
3. Convert to Action Organizer tasks with document link in context

## Step 4: Pull from GitHub Issues (REQUIRES gh CLI)
1. Use `gh` CLI to fetch assigned issues from configured repos
2. Filter by:
   - Assigned to current user
   - State: open
   - Optional: specific repos, labels, milestones

3. Convert GitHub issues to Action Organizer tasks:
   - Map labels to priority (e.g., "priority: high" → 🔴)
   - Include issue number in context
   - Add link to issue

## Step 5: Pull from Slack Reminders (REQUIRES MCP)
1. Use Slack MCP to fetch reminders and saved items
2. Filter by:
   - Active reminders for today/overdue
   - Saved messages marked for follow-up
   - Optional: specific channels or DMs

3. Convert to Action Organizer tasks:
   - Include Slack message link in context
   - Extract any time-related info from reminder

## Step 6: Review & Import
1. Show pulled tasks to the user
2. Ask: "Which tasks do you want to add to today? (all/select/none)"

3. For selected tasks:
   - Add to today's todo file (`YYYY-MM-DD-todo.md`)
   - Maintain source tracking (Jira issue key, Google Doc link)
   - Allow user to adjust priority/timing/estimates

4. Report summary of imported tasks

**Current Status:** This command will be fully functional once Jira, Google Docs, Slack MCPs and GitHub CLI are configured.

**Philosophy:** Bring external commitments into your unified view. Track everything in one place.
