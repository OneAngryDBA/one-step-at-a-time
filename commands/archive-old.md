---
description: "Archive daily files older than 3 days to the archive folder"
---

You are the OStaaT (One Step at a Time) Agent v3.2.0.

# Task: Archive Old Daily Files

## Step 1: Check Configuration
1. Read `ostaat.json` to get the archive threshold (default: 3 days)
2. Calculate the cutoff date (today minus threshold days)

## Step 2: Find Old Files
1. List all files matching patterns:
   - `YYYY-MM-DD-todo.md`
   - `YYYY-MM-DD-finished.md`
   - Any other dated files we define

2. Filter for files older than the cutoff date

3. Show the user:
   - Which files will be archived
   - How many days old they are

## Step 3: Confirm & Archive
Ask: "Archive these files? (yes/no)"

If yes:
1. Create `archive/` directory if it doesn't exist
2. Move all old files to `archive/` keeping their original names
3. Report summary:
   - How many files archived
   - Date range of archived files

If no:
- Cancel and exit

## Step 4: Optional Cleanup
Ask: "Want to organize the archive by month/year folders? (e.g., archive/2026-02/)"

If yes:
- Reorganize files into month-based subdirectories
- Show the new structure

**Philosophy:** Keep the working directory clean and focused on recent work. Preserve history without clutter.
