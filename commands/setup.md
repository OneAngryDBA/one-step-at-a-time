---
description: "Initialize an OStaaT workspace — set up central task management or per-project override"
---

You are the OStaaT (One Step at a Time) Agent v3.2.0.

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
   → Skip to Step 5

## Step 2: Choose Mode

Ask: "Is this repo dedicated to task management, or does it share with code?"

1. **Dedicated** — "This repo is just for OStaaT (tasks, projects, areas)"
   - Set `mode: "dedicated"`, `dataDir: "."`
   - Files will live at the repo root

2. **Shared** — "This repo has code too, keep OStaaT files separate"
   - Set `mode: "shared"`, `dataDir: ".ostaat"`
   - Files will live in a `.ostaat/` subdirectory

## Step 3: Create Workspace Files

Based on the mode, create the following:

### 3a: Config File
Copy the plugin's `assets/config-template.json` to `ostaat.json` in the repo root.
- Set `mode` and `dataDir` based on Step 2 choice
- Set `git.privateRemote: true`

### 3b: Data Files (in dataDir)
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

### 3c: Git Setup
1. If not already a git repo, run `git init`
2. Create/update `.gitignore`:
   - For **dedicated** mode:
     ```
     .DS_Store
     *.log
     .claude/settings.local.json
     ```
   - For **shared** mode, add to existing `.gitignore`:
     ```
     # OStaaT data is version-controlled alongside code
     # If this repo has a PUBLIC remote, consider adding .ostaat/ to .gitignore
     ```

### 3d: Privacy README
If `git.privateRemote` is true and mode is **dedicated**, create a `README.md`:
```markdown
# OStaaT Workspace

Personal task management workspace powered by [One Step at a Time](https://github.com/OneAngryDBA/one-step-at-a-time).

> **Privacy Notice:** This repository contains personal task data, areas of responsibility, and project details. If you push to a remote, ensure it is **PRIVATE**.

## Quick Start

- `/start-day` — Begin your day
- `/dump` — Capture tasks quickly
- `/review-day` — End-of-day review

See the [full documentation](https://github.com/OneAngryDBA/one-step-at-a-time) for all 20 commands.
```

If mode is **shared**, skip the README (the repo likely has its own).

## Step 4: Set as Central Workspace

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
git commit -m "Initialize OStaaT workspace

Mode: dedicated|shared
Central workspace: yes|no

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**Remind the user:**
- "If you push this to a remote, make sure it's a **private** repository. Your task data, areas, and projects are personal."
- "You can now use `/start-day` from **any** Claude Code project to begin."

## Step 5: Per-Project Override

For setting up a local override in a coding project:

Ask: "How should this project's OStaaT work?"

1. **Local workspace** — "Keep tasks separate, just for this project"
   - Ask: dedicated or shared mode (same as Step 2)
   - Create `ostaat.json` in project root
   - Create data files in the appropriate dataDir
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
✅ OStaaT workspace initialized!

  Mode:            dedicated | shared
  Data directory:  . | .ostaat
  Central:         yes | no
  Config:          ostaat.json
  Data files:      AREAS.md, PROJECTS.md

  Next steps:
  - Run /start-day to begin
  - Run /dump to capture tasks
  - Run /new-area to set up areas of responsibility
```

**Philosophy:** Set up once, use everywhere. One step at a time.
