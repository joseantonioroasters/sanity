# Memory Index

This is the project-scoped memory index. Each line is a one-sentence pointer to a memory file in the same directory. Keep it under ~200 lines (the harness may truncate after that).

Place this file at the project memory path your assistant uses (e.g. `~/.claude/projects/<project-slug>/memory/MEMORY.md`).

## Index entries (examples — replace with your own)

- [Always deploy after changes](feedback_always_deploy.md) — Build + deploy after every code change in this project
- [No temporal claims](feedback_no_temporal_claims.md) — Don't say "today", "recent", "last working state" about code; only see snapshots, not a timeline
- [Invoke verify at session start](feedback_invoke_verify_at_start.md) — Walk the verify-latest-source checklist before any file read/edit
- [Project state](project_state.md) — Briefing on architecture, deploy URLs, gotchas

## Notes
- Order doesn't matter; the assistant reads everything linked.
- One line per memory; expand inside the linked file.
- Stale entries hurt more than missing ones — prune ruthlessly.
