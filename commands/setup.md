---
description: "Initialize an OStaaT workspace — set up central task management or per-project override"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Initialize OStaaT Workspace

**Mode: CO-PILOT**

## Step 0: Resolve Workspace Context

Before doing anything, check the current state:

1. Check if `~/.config/ostaat.json` exists (global pointer to central workspace)
2. Check if `ostaat.json` exists in the current directory (already set up here)
3. Check if `.claude/one-step-at-a-time.local.md` exists (per-project override)

**If already set up here:**
- Tell the user: "This directory is already configured as an OStaaT workspace."
- Show current settings summary (mode, dataDir, whether it's the central workspace)
- Check version: if config version < 4.0.0, suggest running `/migrate-tasks` to upgrade
- Ask: "Would you like to reconfigure, or are you looking for something else?"
- If reconfigure, continue to Step 1

**If a central workspace already exists (from ~/.config/ostaat.json) and this is a different directory:**
- Tell the user: "You already have a central workspace at [path]. What would you like to do?"
  1. "Set up a LOCAL override for this project (separate task list)"
  2. "Change the central workspace to THIS directory instead"
  3. "Just point this project to a different workspace"
- Branch accordingly

**If no setup exists anywhere:**
- Continue to Step 1 (fresh setup)

## Step 1: Choose Setup Type

Ask: "How would you like to set up OStaaT?"

1. **"Initialize this directory as my OStaaT workspace"** (default)
   → Continue to Step 2

2. **"Set up a local override for this project"**
   → Skip to Step 6

## Step 2: Choose Mode

Ask: "Is this repo dedicated to task management, or does it share with code?"

1. **Dedicated** — "This repo is just for OStaaT (tasks, projects, areas)"
   - Set `mode: "dedicated"`, `dataDir: "."`
   - Files will live at the repo root

2. **Shared** — "This repo has code too, keep OStaaT files separate"
   - Set `mode: "shared"`, `dataDir: ".ostaat"`
   - Files will live in a `.ostaat/` subdirectory

## Step 3: Energy Dimensions (Optional)

Ask: "OStaaT can track energy types for tasks (e.g., 🧠 Deep focus, ⚡ Admin, 🎨 Creative). Want to set up energy labels now? (You can always add them later)"

**If yes:**
1. Explain: "Energy labels help categorize tasks by the type of mental energy they need. You can define your own labels."
2. Show defaults as suggestions:
   - `🧠 Deep` — Deep focus, complex problem solving
   - `⚡ Admin` — Mechanical, administrative tasks
   - `🎨 Creative` — Design, writing, ideation
   - `🤝 Social` — Meetings, calls, collaboration
3. Ask: "Use these defaults, customize them, or skip for now?"
   - **Use defaults** → set `energy.dimensions` to the four defaults, `energy.enabled: false` (Phase 2 enables enforcement)
   - **Customize** → ask for each label (emoji + name + description), one at a time
   - **Skip** → leave `energy.dimensions` empty

**If no:**
- Leave `energy.dimensions` empty, proceed

Note: Energy labels are stored but NOT enforced until Phase 2. They appear as optional metadata on tasks.

## Step 4: Create Workspace Files

Based on the mode, create the following:

### 4a: Config File
Copy the plugin's `assets/config-template.json` to `ostaat.json` in the repo root.
- Set `mode` and `dataDir` based on Step 2 choice
- Set `git.privateRemote: true`
- Set `energy.dimensions` based on Step 3 (if configured)
- Set `tasks.nextId: 1`

### 4b: Data Files (in dataDir)
Create:
- `AREAS.md` with a starter template:
  ```markdown
  # Areas of Responsibility

  _No areas created yet. Use `/new-area` to add your first area._
  ```
- `PROJECTS.md` with a starter template:
  ```markdown
  # Active Projects

  _No projects created yet. Use `/new-project` to start your first project._
  ```

### 4c: Task Store (in dataDir)
Create the `tasks/` directory (or `{tasks.storeDir}/`) with:

- `INDEX.md`:
  ```markdown
  # Task Index

  | ID | Status | Summary | Tag |
  |----|--------|---------|-----|
  ```

- `inbox.md`:
  ```markdown
  # Inbox
  ```

- `ready.md`:
  ```markdown
  # Ready
  ```

- `in-progress.md`:
  ```markdown
  # In Progress
  ```

- `waiting.md`:
  ```markdown
  # Waiting
  ```

- `paused.md`:
  ```markdown
  # Paused
  ```

### 4d: Templates Directory
Create `templates/tasks/` directory (if it doesn't already exist).

### 4e: Git Setup
1. If not already a git repo, run `git init`
2. Create/update `.gitignore`:
   - For **dedicated** mode:
     ```
     .DS_Store
     *.log
     .ostaat.lock
     .claude/settings.local.json
     ```
   - For **shared** mode, add to existing `.gitignore`:
     ```
     .ostaat.lock
     # OStaaT data is version-controlled alongside code
     # If this repo has a PUBLIC remote, consider adding .ostaat/ to .gitignore
     ```

### 4f: Privacy README
If `git.privateRemote` is true and mode is **dedicated**, create a `README.md`:
```markdown
# OStaaT Workspace

Personal task management workspace powered by [One Step at a Time](https://github.com/OneAngryDBA/one-step-at-a-time).

> **Privacy Notice:** This repository contains personal task data, areas of responsibility, and project details. If you push to a remote, ensure it is **PRIVATE**.

## Quick Start

- `/start-day` — Begin your day
- `/dump` — Capture tasks quickly
- `/review-day` — End-of-day review
- `/list-tasks` — See all open tasks

See the [full documentation](https://github.com/OneAngryDBA/one-step-at-a-time) for all 26 commands.
```

If mode is **shared**, skip the README (the repo likely has its own).

## Step 5: Set as Central Workspace

Ask: "Set this as your central OStaaT workspace? (All Claude Code projects will use this by default)"

- **Yes** (default): Write `~/.config/ostaat.json`:
  ```json
  {
    "workspace": "/absolute/path/to/this/directory"
  }
  ```
- **No**: Skip. The user can set it later or use per-project overrides.

Then create the initial commit:
```
git add -A
git commit -m "Initialize OStaaT v4.0 workspace

Mode: dedicated|shared
Central workspace: yes|no
Task store: tasks/ with INDEX.md + status files
Energy dimensions: configured|skipped

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**Remind the user:**
- "If you push this to a remote, make sure it's a **private** repository. Your task data, areas, and projects are personal."
- "You can now use `/start-day` from **any** Claude Code project to begin."

## Step 6: Per-Project Override

For setting up a local override in a coding project:

Ask: "How should this project's OStaaT work?"

1. **Local workspace** — "Keep tasks separate, just for this project"
   - Ask: dedicated or shared mode (same as Step 2)
   - Create `ostaat.json` in project root
   - Create data files in the appropriate dataDir (including tasks/ directory)
   - Write `.claude/one-step-at-a-time.local.md`:
     ```yaml
     ---
     workspace: local
     ---

     This project has its own OStaaT workspace.
     ```

2. **Point to a different workspace** — "Use a specific directory instead of the central one"
   - Ask for the path
   - Write `.claude/one-step-at-a-time.local.md`:
     ```yaml
     ---
     workspace: /path/to/other/workspace
     ---

     This project uses a custom OStaaT workspace.
     ```

Ask the user if they want to add any notes to the markdown body (e.g., "This project tracks client deliverables separately").

## Summary

After setup, show:
```
✅ OStaaT v4.0 workspace initialized!

  Mode:            dedicated | shared
  Data directory:  . | .ostaat
  Central:         yes | no
  Config:          ostaat.json
  Task store:      tasks/ (INDEX.md + 5 status files)
  Energy labels:   configured (4 labels) | skipped
  Data files:      AREAS.md, PROJECTS.md

  Next steps:
  - Run /start-day to begin
  - Run /dump to capture tasks
  - Run /new-area to set up areas of responsibility
  - Run /list-tasks to see your task backlog
```

**Philosophy:** Set up once, use everywhere. One step at a time.
