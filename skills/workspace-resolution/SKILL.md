---
name: workspace-resolution
description: This skill should be used BEFORE any OStaaT command reads or writes data files (AREAS.md, PROJECTS.md, daily todo/finished files, scheduled/, archive/, templates/). It resolves which workspace directory to use based on per-project overrides or the global central workspace, and enforces workspace locking to prevent concurrent write conflicts. It should activate automatically when any OStaaT command runs. It should NOT be used for general questions or conversations that don't involve OStaaT data files.
---

# Workspace Resolution Skill

**Every OStaaT command must resolve the workspace AND check the lock before reading or writing any data.**

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

---

## Workspace Locking

After resolving the workspace, **check and acquire the lock before any write operation.**

### Lock file location
`{workspace_root}/.ostaat.lock`

### Lock file format
```json
{
  "session": "<unique session identifier>",
  "command": "<command that acquired the lock>",
  "acquired": "<ISO 8601 timestamp>",
  "cwd": "<working directory of the session that acquired the lock>"
}
```

The session identifier should be constructed from available context: the current working directory path + a timestamp of when this Claude session started (or the first lock acquisition). The goal is to distinguish "this Claude session" from "another Claude session."

### Lock check algorithm

**For WRITE commands** (`/start-day`, `/dump`, `/brain-dump`, `/add-task`, `/review-day`, `/refine`, `/archive-old`, `/new-project`, `/update-project`, `/link-task`, `/reopen-project`, `/review-projects`, `/new-area`, `/update-area`, `/review-areas`, `/pull`, `/allocate-time`):

1. Read `{workspace_root}/.ostaat.lock`
2. **If no lock file exists** → acquire the lock (create the file), proceed with command
3. **If lock file exists:**
   a. Parse the JSON
   b. Check if the lock is **stale**: if `acquired` timestamp is more than 10 minutes ago, treat as stale
   c. Check if the lock belongs to **this session** (same `session` identifier): if yes, proceed (re-entrant)
   d. If the lock is **stale** → warn the user, auto-break and re-acquire:
      ```
      ⚠️ Stale lock detected (acquired 25 minutes ago by another session).
      Auto-breaking stale lock and proceeding.
      ```
   e. If the lock is **active** (not stale, different session) → **BLOCK**:
      ```
      🔒 Workspace is locked by another session.
      Locked by: <command> at <time> (from <cwd>)
      Lock age: <N> minutes

      Options:
      - Wait and try again in a few minutes
      - Force break the lock: /ostaat-unlock --force
      - If the other session is done, it should release automatically
      ```
      **Do NOT proceed with the command.**

**For READ-ONLY commands** (`/list-projects`, `/list-areas`, `/ostaat-help`):
- Do NOT acquire a lock — these are safe to run concurrently
- Do NOT check for a lock — reads are always allowed

### Lock lifecycle

1. **Acquire** — at the start of any write command, after workspace resolution
2. **Hold** — throughout the command execution (Claude is reading files, asking user questions, writing files)
3. **Release** — after the command completes (including git commit if applicable)
4. **Auto-expire** — 10 minutes after acquisition, the lock is considered stale

### Releasing the lock

At the END of every write command, after all file writes and git commits are complete:
1. Read `{workspace_root}/.ostaat.lock`
2. Verify it belongs to this session
3. Delete the lock file
4. Do NOT mention the lock release to the user (silent operation)

If the lock file doesn't exist or belongs to another session at release time, do nothing (another session may have force-broken it).

### Refreshing the lock

If a write command involves multiple user interactions (e.g., `/start-day` asks several questions), **refresh the lock** by updating the `acquired` timestamp before each write operation. This prevents the lock from going stale during long interactive sessions.

---

## Using the Resolved Workspace

Once resolved, ALL file operations use the workspace path:

| File | Path |
|------|------|
| Config | `{workspace_root}/ostaat.json` (always at root, not in dataDir) |
| Lock | `{workspace_root}/.ostaat.lock` |
| AREAS.md | `{workspace}/{dataDir}/AREAS.md` |
| PROJECTS.md | `{workspace}/{dataDir}/PROJECTS.md` |
| Daily todo | `{workspace}/{dataDir}/YYYY-MM-DD-todo.md` |
| Daily finished | `{workspace}/{dataDir}/YYYY-MM-DD-finished.md` |
| Scheduled | `{workspace}/{dataDir}/scheduled/` |
| Archive | `{workspace}/{dataDir}/archive/` |
| Project archive | `{workspace}/{dataDir}/projects-archive/` |
| Templates | `{workspace}/{dataDir}/templates/` |

**Note:** `ostaat.json` and `.ostaat.lock` always live at the workspace root directory, NOT inside `dataDir`. This keeps config and locks separate from data in shared mode.

## Git Operations

When `git.autoCommit` is enabled:
- Change directory to the workspace root before git operations
- The workspace root is the git repo, not necessarily the current project
- This means commits happen in the OStaaT workspace repo, not the current coding project
- **The `.ostaat.lock` file should be in `.gitignore`** — never commit lock files

## Hard Rules

- **NEVER read or write OStaaT data files without resolving the workspace first**
- **NEVER write to OStaaT data files without checking/acquiring the lock first**
- **NEVER create data files in the current directory** unless it IS the resolved workspace
- **NEVER assume the workspace is the current directory** — always resolve
- **NEVER proceed with a write command if the lock is held by another active session**
- **ALWAYS release the lock when a write command completes**
- **ALWAYS refresh the lock timestamp during long interactive commands**
- **If resolution fails, tell the user to run /setup** — don't guess
