---
name: workspace-resolution
description: This skill should be used BEFORE any OStaaT command reads or writes data files (AREAS.md, PROJECTS.md, daily todo/finished files, scheduled/, archive/, templates/). It resolves which workspace directory to use based on per-project overrides or the global central workspace. It should activate automatically when any OStaaT command runs. It should NOT be used for general questions or conversations that don't involve OStaaT data files.
---

# Workspace Resolution Skill

**Every OStaaT command must resolve the workspace before reading or writing any data.**

## Resolution Algorithm

Execute these steps in order. Stop at the first match:

### Step 1: Check for per-project override
Read `.claude/one-step-at-a-time.local.md` in the current working directory.

- If found with `workspace: local` in YAML frontmatter:
  - Read `ostaat.json` from the current working directory
  - Use `dataDir` from that config (default: `"."`)
  - **Workspace = {current_directory}/{dataDir}**

- If found with `workspace: /some/path` in YAML frontmatter:
  - Read `ostaat.json` from that path
  - Use `dataDir` from that config
  - **Workspace = {specified_path}/{dataDir}**

### Step 2: Check for global central workspace
Read `~/.config/ostaat.json`.

- If found, read the `workspace` field
- Read `ostaat.json` from that workspace path
- Use `dataDir` from that config
- **Workspace = {global_workspace_path}/{dataDir}**

### Step 3: No workspace found
If neither exists:
- Tell the user: "No OStaaT workspace found. Run `/setup` to initialize one."
- Do NOT proceed with the command
- Do NOT create files in the current directory without explicit setup

## Using the Resolved Workspace

Once resolved, ALL file operations use the workspace path:

| File | Path |
|------|------|
| Config | `{workspace_root}/ostaat.json` (always at root, not in dataDir) |
| AREAS.md | `{workspace}/{dataDir}/AREAS.md` |
| PROJECTS.md | `{workspace}/{dataDir}/PROJECTS.md` |
| Daily todo | `{workspace}/{dataDir}/YYYY-MM-DD-todo.md` |
| Daily finished | `{workspace}/{dataDir}/YYYY-MM-DD-finished.md` |
| Scheduled | `{workspace}/{dataDir}/scheduled/` |
| Archive | `{workspace}/{dataDir}/archive/` |
| Project archive | `{workspace}/{dataDir}/projects-archive/` |
| Templates | `{workspace}/{dataDir}/templates/` |

**Note:** `ostaat.json` always lives at the workspace root directory, NOT inside `dataDir`. This keeps config separate from data in shared mode.

## Git Operations

When `git.autoCommit` is enabled:
- Change directory to the workspace root before git operations
- The workspace root is the git repo, not necessarily the current project
- This means commits happen in the OStaaT workspace repo, not the current coding project

## Hard Rules

- **NEVER read or write OStaaT data files without resolving the workspace first**
- **NEVER create data files in the current directory** unless it IS the resolved workspace
- **NEVER assume the workspace is the current directory** — always resolve
- **If resolution fails, tell the user to run /setup** — don't guess
