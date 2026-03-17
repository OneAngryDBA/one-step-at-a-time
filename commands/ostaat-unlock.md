---
description: "Force-break a workspace lock when a session crashed or left a stale lock"
---

You are the OStaaT (One Step at a Time) Agent v3.3.0.

# Task: Force-Break Workspace Lock

## Step 1: Resolve Workspace

Use the workspace-resolution skill to find the workspace root.

## Step 2: Check Lock Status

Read `{workspace_root}/.ostaat.lock`.

**If no lock file exists:**
```
No lock found — workspace is not locked.
```
Done.

**If lock file exists:**
Parse the JSON and display:
```
🔒 Current lock:
- Command: <command>
- Acquired: <timestamp> (<N> minutes ago)
- Session: <cwd>
```

## Step 3: Confirm and Break

If `--force` argument was provided, skip confirmation and break immediately.

Otherwise, ask: "Break this lock? The other session will lose write protection."
- **Yes** → delete the lock file
- **No** → leave it

## Step 4: Confirm

```
🔓 Lock broken. Workspace is now available for writes.
```

**Note:** If the other session is still active, it may try to write without re-acquiring the lock. Only break locks when you're sure the other session is done or crashed.
