---
name: sosp
description: >
  Start of Session Protocol (SOSP). Mandatory pre-flight verification of the
  project's sources of truth — git branches and worktrees, deployment
  topology, active architecture wiring, untracked critical files, and
  deployed bundle contents — before reading, editing, or deploying code in
  any existing codebase. Always invoke at the very start of any coding task
  in an existing project; no exceptions, no shortcuts. Especially required
  when inside a git worktree, on a Claude-managed branch, when the user
  references a URL or deployment, when multiple architectures or branches
  co-exist in the repo, when the user mentions "today" / "recent" /
  "yesterday" about code state, or when the user says "fix", "change",
  "add", "deploy", "revert", "look at", "debug", "diagnose", "review",
  "check", or "what's wrong with the app." Pairs with EOSP — SOSP reads
  what EOSP wrote and verifies it against current reality.
  Skip only for greenfield projects with no prior state.
---

# SOSP — Start of Session Protocol

The pre-flight checklist for any coding task in an existing codebase.
Editing or deploying based on assumptions is worse than doing nothing.
Run this **before touching any file.**

Pairs with **EOSP** (End of Session Protocol). SOSP reads what EOSP wrote
into memory and verifies it against current reality (git, deployment,
imports, runtime) before letting any work proceed.

## Why this exists

These failure modes are silent, common, and expensive to undo:

- Editing a file in a stale git worktree while newer commits live elsewhere
- Deploying to a preview URL while the user tests production
- Reverting to an "old working state" that is actually internally inconsistent
- Treating a branch name as the architecture (a `foo-vision` branch may contain `bar` code)
- Losing functionality by `git checkout`ing past untracked load-bearing files
- Blaming browser cache without verifying which bundle the URL actually serves
- Making temporal claims about code ("yesterday's state", "what was working") without a real timeline anchor

## Checklist

### 1. Git source-of-truth

```bash
pwd
git worktree list
git branch --show-current
git status --short
git log --oneline --all -10
```

Answer for yourself:
- Am I in a worktree? Where is the parent repo?
- Does another branch have newer commits than the one I'm on?
- Are there uncommitted changes I might overwrite?

### 2. Deployment topology

When the user references any URL, identify which build it serves:

```bash
curl -s "<production-url>"  | grep -oE 'index-[^"]+\.js'
curl -s "<branch-url>"      | grep -oE 'index-[^"]+\.js'
ls dist/assets/ 2>/dev/null
```

- Which URL is production, which is preview/branch?
- Does the current branch deploy to production or only to a preview?
- Does the local build hash match what the user is testing against?

### 3. Active architecture — from imports, not filenames

The entry point's `import` statements are the only truth about what code is
actually running. Branch names and file existence both lie.

```bash
grep -E "^import|require\(" <entry-point>     # what's actually wired in
```

Cross-check against the deployed bundle:

```bash
curl -s <url>/<bundle> | grep -oE "<architecture-marker>" | sort -u
```

If two parallel architectures co-exist (engine A and engine B), confirm
which one the entry point imports right now — even if filenames or the
branch name suggest the other.

### 4. Untracked critical files

```bash
git ls-files <critical-dirs>
git status -uall <critical-dirs>
```

If an `import`ed file appears as `??` (untracked), flag it immediately:

- It has no git history to revert to
- A careless `git checkout` deletes it
- It exists only in this working tree
- Recovery requires the user, not git

### 5. Deployed bundle verification (when the user reports a runtime bug)

Before blaming browser cache, network, or proposing a fix:

```bash
curl -sI <url>                                        # response headers
curl -s <url> | grep -oE 'index-[^"]+\.js'            # served bundle
curl -s <url>/<path-to-bundle> | grep -oE "<marker>"  # confirm contents
```

Compare to local `dist/`. A mismatch means the deploy didn't reach the URL
the user is testing.

## Anti-patterns to refuse

- Reverting to a historical commit without verifying the snapshot is
  internally consistent with the file's current callers and callees
- Describing snapshots by time ("the state before today", "the recent
  default", "what was working last week") rather than by content
- Blaming browser cache before running `curl -sI` on the deployment
- Assuming the current working directory is the source of truth
- Reading a file from memory or from an earlier read in the same session —
  always re-read immediately before editing
- Treating commit timestamps as a personal timeline ("this was today" /
  "this was before this session"): they're labels on snapshots, not a
  wall-clock you can navigate

## Resolve ambiguity by asking, not guessing

If after the checklist it remains unclear which file, branch, or deployment
is the target, stop and ask:

> "I see version X in the worktree on branch A, version Y in main, and
> version Z on the deployed URL. Which should I edit, and which should I
> treat as the source of truth?"

Never default to the current working directory. Never default to "the most
recent commit." Never default to a guess about what the user remembers
working.

## Decision tree

```
Start of coding task in existing codebase
        │
        ▼
Run Checklist §1 (git topology)
        │
        ├─ Multiple worktrees / divergent branches? ──► §1 resolution
        │
        ▼
User mentioned a URL or deployment?
        │
        ├─ Yes ──► Run Checklist §2 (deployment topology)
        │
        ▼
Run Checklist §3 (imports = active architecture)
        │
        ▼
Run Checklist §4 (untracked critical files)
        │
        ▼
User reports runtime bug?
        │
        ├─ Yes ──► Run Checklist §5 (bundle verification)
        │
        ▼
Still ambiguous about what to edit? ──► Ask, don't guess.
        │
        ▼
Proceed with the edit, then deploy and verify.
```

## Non-negotiable rules

1. Always run this checklist before any file read or edit in an existing codebase.
2. Never assume the current working directory is the source of truth.
3. Always read the target file immediately before editing it.
4. If file content surprises you (missing features, different structure
   than expected) — stop. You are probably reading the wrong version.
5. Before deploying, confirm the build is running from the same directory
   whose source files you edited.
6. Before claiming a deployment is or isn't serving your changes, curl the
   bundle hash and verify by content.
