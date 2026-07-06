# Changelog

All notable changes to **One Step at a Time (OStaaT)** are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## [4.0.0] - 2026-07-06

Major release. The headline change is a **persistent task store** — tasks are now
durable, first-class objects rather than lines inside a daily file. Everything else
builds on top of that.

### Added
- **Persistent task store** — tasks live in a durable store (sharded by status) with
  stable task IDs (`t-NNN`), subtasks, and explicit lifecycle states. Daily files now
  reference tasks instead of owning them.
- **Unified templates** — reusable task/project templates via `/new-from-template`.
- **Area hierarchy** — path-based sub-areas under areas of responsibility.
- **Energy-aware planning (Phase 2)** — an energy calendar with weekly defaults and
  daily overrides, area-scoped time blocks, and constraint-based scheduling with
  infeasibility detection (`/allocate-time`, `/energy-calendar`).
- **Focus modes** — `/panic` for full crisis triage across tasks, projects, areas, and
  deadlines; `/hyperfocus` to lock onto one project or task and defer everything else.
- **`/upgrade`** — migrate an existing v3.x workspace to the v4.0 structure.
- **`TEST-CHECKLIST.md`** — verification checklist for the v4.0 release.

### Changed
- Grew from **22 to 30 slash commands**.
- Daily workflow (`/start-day`, `/review-day`) now reads from and writes to the
  persistent task store rather than storing tasks solely in daily files.
- Updated design docs: `SYSTEM-ARCHITECTURE.md`, `TASKS-DESIGN.md` (new),
  `ENERGY-DESIGN.md` (new), `AREAS-DESIGN.md`, `USAGE.md`.

### Migration
- Existing users: run **`/upgrade`** to migrate a v3.x workspace to v4.0. It handles
  config, structure, and data migration into the persistent task store.

## [3.3.0] - 2026-03

Previous public release: daily-file-based task management with 22 slash commands,
6 proactive skills, workspace locking, area management with recurring items, and
automatic daily rollover. (Releases prior to 4.0.0 were not tracked in this changelog.)

[4.0.0]: https://github.com/OneAngryDBA/one-step-at-a-time/releases/tag/v4.0.0
