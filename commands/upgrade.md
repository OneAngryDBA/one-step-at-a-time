---
description: "Upgrade an OStaaT workspace from any previous version to the latest — handles config, structure, and data migration"
---

You are the OStaaT (One Step at a Time) Agent v4.0.0.

**⚠️ LOCKING: This command writes to the workspace. You MUST acquire the workspace lock before writing any files and release it when done. See the workspace-resolution skill for the full locking protocol. If the lock is held by another session, do NOT proceed — inform the user.**

# Task: Upgrade OStaaT Workspace

**Mode: CO-PILOT**

This command detects the current workspace version and walks through each upgrade step needed to reach v4.0.0. It handles config changes, directory creation, file renames, and data migration.

## Step 0: Detect Current Version

1. Resolve the workspace (workspace-resolution skill)
2. Read `ostaat.json` (or `todo-config.json` for pre-3.1 workspaces)
3. Check for version field:
   - If `todo-config.json` exists → **v3.0.x** (pre-rename)
   - If `ostaat.json` exists with `"version": "3.1.x"` → **v3.1.x**
   - If `ostaat.json` with `"version": "3.2.x"` → **v3.2.x**
   - If `ostaat.json` with `"version": "3.3.x"` → **v3.3.x**
   - If `ostaat.json` with `"version": "4.0.x"` → **already current**
   - If no config found → suggest `/setup` instead

4. Show current state:
   ```
   📋 OStaaT Upgrade

   Current version: 3.1.0
   Target version:  4.0.0
   Workspace:       /path/to/workspace

   Upgrades needed:
   ✅ v3.0 → v3.1 (already done)
   ☐ v3.1 → v3.2 — Add help system references
   ☐ v3.2 → v3.3 — Add workspace locking config
   ☐ v3.3 → v4.0 — Persistent task store, templates, area hierarchy

   Proceed? (Each step will be explained before applying)
   ```

5. If already on v4.0.0: "Your workspace is already on the latest version."

---

## Upgrade: v3.0.x → v3.1.0

**What changed:** Renamed from OStaaP to OStaaT. Config file renamed. Workspace architecture added with `/setup` command and workspace-resolution skill.

### Steps:

1. **Rename config file**
   - Rename `todo-config.json` → `ostaat.json`
   - If both exist, ask which to keep

2. **Add workspace fields to config**
   ```json
   {
     "version": "3.1.0",
     "mode": "dedicated",
     "dataDir": "."
   }
   ```
   - Ask: "Is this a dedicated OStaaT workspace, or shared with code?"
   - Set `mode` and `dataDir` accordingly

3. **Add git.privateRemote**
   ```json
   "git": {
     "privateRemote": true
   }
   ```

4. **Set up central workspace pointer** (if not already set)
   - Ask: "Set this as your central OStaaT workspace?"
   - If yes, write `~/.config/ostaat.json` with workspace path

5. **Add .ostaat.lock to .gitignore**

6. **Update version** in config to `"3.1.0"`

```
✅ Upgraded to v3.1.0
   - Config renamed: todo-config.json → ostaat.json
   - Workspace mode: dedicated
   - Central workspace: set
```

---

## Upgrade: v3.1.x → v3.2.0

**What changed:** Added help system — `/ostaat-help` command and help skill. Added USAGE.md documentation. Command count went from 20 to 21, skills from 5 to 6.

### Steps:

1. **No config changes needed** — this was a feature addition, not a config change

2. **Update version** in config to `"3.2.0"`

```
✅ Upgraded to v3.2.0
   - No config changes needed
   - New command available: /ostaat-help
   - New docs available: docs/USAGE.md
```

Note: The help command and skill come from the plugin itself — no workspace changes required.

---

## Upgrade: v3.2.x → v3.3.0

**What changed:** Added workspace locking to prevent concurrent write conflicts. Lock file `.ostaat.lock` with session tracking, 10-minute auto-expiry, and `/ostaat-unlock` command.

### Steps:

1. **Ensure .ostaat.lock is in .gitignore**
   - Read `.gitignore`
   - If `.ostaat.lock` is not listed, add it
   - This prevents lock files from being committed

2. **No config schema changes** — locking is handled by the workspace-resolution skill

3. **Update version** in config to `"3.3.0"`

```
✅ Upgraded to v3.3.0
   - .ostaat.lock added to .gitignore
   - Workspace locking now active on all write commands
   - New command available: /ostaat-unlock
```

---

## Upgrade: v3.3.x → v4.0.0

**What changed:** Persistent task store inverts the data model. Tasks live in a sharded `tasks/` directory as the source of truth. Daily files become generated views. Task IDs, lifecycle states, subtasks, unified templates, area hierarchy, energy dimensions (stored, not enforced).

This is the most significant upgrade. It has three phases:

### Phase 1: Config migration

Add new config sections to `ostaat.json`:

```json
{
  "version": "4.0.0",
  "tasks": {
    "enabled": true,
    "nextId": 1,
    "showInboxAtStartDay": true,
    "autoSelectInProgress": true,
    "storeDir": "tasks"
  },
  "energy": {
    "enabled": false,
    "dimensions": []
  },
  "templates": {
    "folder": "templates/tasks",
    "promptOnAreaDue": true
  }
}
```

Also add to existing `areas` section:
```json
"areas": {
  "hierarchyEnabled": true,
  "maxDepth": 2
}
```

Also add new commands to `git.commitOnActions`:
```json
"/update-task",
"/new-from-template",
"/migrate-tasks",
"/upgrade"
```

**Ask about energy dimensions:**
"OStaaT v4 supports energy labels on tasks (e.g., 🧠 Deep focus, ⚡ Admin). Want to configure them now?"
- If yes, follow the energy setup flow from `/setup` Step 3
- If no, leave `energy.dimensions` empty (can be added later)

### Phase 2: Directory creation

Create the task store directory structure:

```
{dataDir}/tasks/
├── INDEX.md         # Empty table header
├── inbox.md         # Empty with header
├── ready.md         # Empty with header
├── in-progress.md   # Empty with header
├── waiting.md       # Empty with header
└── paused.md        # Empty with header
```

Create `templates/tasks/` directory if it doesn't exist.

### Phase 3: Task migration

Ask: "Want to migrate existing tasks from your daily files into the task store now?"

- **Yes** → Run the `/migrate-tasks` flow (scan daily files, extract tasks, assign IDs, write to store)
- **Later** → "You can run `/migrate-tasks` anytime to bring existing tasks into the store. New tasks will go to the store automatically."
- **Skip** → "Starting fresh. The task store is empty and ready for new tasks."

### Phase 4: Summary

```
✅ Upgraded to v4.0.0

  Config:
  - Added tasks, energy, templates sections
  - Added areas.hierarchyEnabled
  - Energy dimensions: configured (4 labels) | skipped

  New directories:
  - tasks/ — persistent task store with 5 status files + INDEX.md
  - templates/tasks/ — unified template directory

  Task migration: completed (15 tasks) | deferred | skipped

  New commands available:
  - /list-tasks — query all open tasks
  - /update-task — change task status
  - /new-from-template — create from templates
  - /migrate-tasks — import from daily files (run anytime)

  Changed behavior:
  - /start-day now reads from the task store (no more rolling forward)
  - /dump, /add-task write to the task store with auto IDs
  - /review-day marks done in the task store
  - Daily files are generated views, not the source of truth

  Next: Run /start-day to try the new focus selection flow.
```

---

## Running Multiple Upgrades

If the workspace is multiple versions behind, each upgrade runs in sequence:

```
📋 Upgrading from v3.0.0 to v4.0.0 (4 steps)

Step 1/4: v3.0 → v3.1 ...
✅ Done

Step 2/4: v3.1 → v3.2 ...
✅ Done

Step 3/4: v3.2 → v3.3 ...
✅ Done

Step 4/4: v3.3 → v4.0 ...
(interactive — energy config, task migration)
✅ Done

🎉 Workspace fully upgraded to v4.0.0!
```

Each step applies atomically. If something fails, the version stays at the last successful step.

---

## Git Commit

If `git.autoCommit` enabled, commit after all upgrades complete:
```
Upgrade OStaaT workspace: v{{from}} → v{{to}}

{{list of changes per version step}}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Safety

- **Non-destructive:** No files are deleted. Config fields are added, not replaced.
- **Reversible:** Git history preserves the pre-upgrade state.
- **Incremental:** Each version step is independent. Partial upgrades are valid.
- **Idempotent:** Running upgrade on a current version does nothing.

**Philosophy:** Upgrades should be safe, transparent, and explainable. Show what will change before changing it. Never lose data.
