# Session Protocols — Working Together

A shared playbook between a user and an agentic coding assistant using the three-tool system:
**EOSP** (end-of-session memory write) + **SOSP** (start-of-session verification) + **Codegraph** (persistent code index).

Scoped to **modifications on existing codebases**, not greenfield projects.

---

## The three tools at a glance

| Tool | Trigger | What it does | Direction |
|---|---|---|---|
| **EOSP** (skill) | You type exactly `EOSP` | Writes a dense state snapshot into project memory files | Writes |
| **SOSP** (skill) | Auto-fires on most coding requests in existing codebases | Verifies git topology, deployment URL, active imports, untracked files, deployed bundle | Reads + verifies |
| **Codegraph** (`codegraph-auto-init` skill) | Auto-init via UserPromptSubmit hook in qualifying repos | Builds persistent SQL index of symbols, imports, call graph | Indexes |

Together: EOSP writes the story → Codegraph indexes the structure → SOSP cross-checks both against reality before the assistant acts.

**Both SOSP and EOSP are now 1:1 with their skills.** Generic enforcement lives in the skill descriptions themselves (global, applies to every project). Project memory is reserved for **project-specific landmines** only.

---

## Scenarios — what you say, what the assistant does

### A. Start of a new session on an existing codebase (the main case)

**You say** (any of these is enough):
- Just state your goal: *"Fix the camera permission on Android"* / *"Change the loader color to blue"* / *"Deploy the latest"*
- Or explicitly: *"Run SOSP first"* / *"Verify source of truth before starting"*

The `sosp` skill description should auto-fire on words like *fix, change, add, deploy, revert, look at, debug, diagnose, review, check, what's wrong*. The explicit form guarantees it.

**You provide upfront (saves several turns):**
- The deployment URL you're testing on right now
- One sentence on the last behavior you remember working
- Branch preference if you have one

**The assistant does:**
1. Read project memory (`MEMORY.md` + the files it links — including any `project_landmines.md` for this project)
2. Read `working.md` if it exists at the project root
3. Run `sosp` skill checklist: git topology → deployment URL bundle hashes → entry-point imports → untracked critical files
4. Query codegraph (if initialized) for active import graph instead of grepping
5. Report a one-screen situation summary and a plan
6. Wait for your confirmation OR proceed if the plan is clearly tactical

---

### B. Mid-session — you sense drift

Don't wait until things get worse. These short phrases re-anchor the assistant:

| You say | The assistant does |
|---|---|
| *"Verify what's deployed"* | Re-curl the URL, grep bundle markers, compare to local build |
| *"Re-check entry-point imports"* | Re-grep imports, state active architecture |
| *"What URL is your deploy landing on?"* | State explicitly: production vs preview alias vs unique hash URL |
| *"Stop. Re-verify."* | Halt, re-run full SOSP checklist before continuing |
| *"You're drifting. Show me what you actually know."* | List my current assumptions with evidence; flag anything I'm guessing at |

Cost of calling these is low. Cost of letting drift accumulate is high.

---

### C. Ending a session cleanly

**You say:** `EOSP` (exact phrase, anywhere in a message)

**The assistant does:** invoke the `eosp` skill, which writes a structured snapshot into project memory:
- Current branch + uncommitted state
- What was changed this session and why
- What was deployed (URL + bundle hash)
- What's pending / what to try next
- Gotchas to remember

Then I stop and report the file paths I wrote.

---

### D. Switching to a different codebase

**You say:** [open a new conversation in a different working directory]

**Automatic:** project-scoped memory path changes based on cwd. Codegraph re-checks for `.codegraph/` in the new repo. The `sosp` skill applies globally — works on any project without setup.

**You should still:** name the project in your first message so I read the right memory.

---

### E. New / greenfield project

**You say:** *"Starting a new project"* or describe greenfield work explicitly.

**The assistant does:** skip SOSP entirely (nothing to verify — the skill description says "Skip only for greenfield projects with no prior state"). Codegraph won't init until source file count crosses ~30. EOSP is still useful for tracking decisions made.

---

## Quick-reference card

| Intent | Phrase | What fires |
|---|---|---|
| Start work on existing codebase | "fix / change / add / deploy / revert ..." | SOSP auto-fire |
| Force full verification | "Run SOSP" / "Verify before continuing" | SOSP explicit |
| Re-check deployment | "What's at \<URL\>?" | Curl + bundle marker grep |
| Re-check architecture | "What does the entry-point actually import?" | grep entry-point or codegraph |
| Re-check git source | "What's the git topology?" | git worktree + branch + log |
| Stop drift | "Stop. Re-verify." | Halt + re-run SOSP |
| Close session | "EOSP" | EOSP writes snapshot |
| Resume next session | (anything in scenario A) | SOSP reads what EOSP wrote |

---

## What you do NOT need to say (already enforced)

**Global (enforced by skill descriptions):**
- Invoke SOSP at session start — the `sosp` skill self-triggers on any coding request in an existing codebase
- Invoke EOSP on session close — the `eosp` skill triggers on the exact phrase `EOSP`

**Project-scoped (enforced via project memory):**
- Project-specific deploy commands (e.g. build + deploy after every change)
- Project-specific conventions (file formats, naming, gotchas)
- Project-specific landmines (parallel architectures, locked URLs, untracked critical files)

**Universal feedback rules (in project memory, but generic enough to drop into any project):**
- **No temporal claims about code** — describe snapshots by content, not by "today" / "recent" / "last working"

If the assistant violates any of these, call it out — it means a rule isn't firing.

---

## Anti-patterns the assistant should refuse

If you see the assistant about to do any of these, interrupt with **"Verify first"**:

- Revert to a historical commit without checking internal consistency with current callers/callees
- Blame browser cache before running `curl -sI` on the URL
- Describe a snapshot by *when* it was instead of *what* it contains
- Claim a deploy serves the URL you're testing without confirming the bundle hash
- Edit a file based on memory of an earlier read in the same session — should always re-read first

---

## Three-tool failure modes to watch for

| Failure | How to catch it |
|---|---|
| **Codegraph stale** (~1 sec watcher lag after edits) | If the assistant just edited then queries, results may lag. Always trust reality (git, curl) when codegraph and reality disagree. |
| **Memory drift** (EOSP wrote wrong state) | Override directly: *"Memory is wrong about X. Here's the truth: ..."* — the assistant should update the relevant file. |
| **Skill trigger missed** (SOSP didn't auto-fire on subtle phrasing) | Use explicit *"Run SOSP"* instead of relying on heuristic match. |
| **Three layers compound** (codegraph stale + memory wrong + assistant trusts majority) | Force re-verify against reality: *"Curl the URL. Read the entry-point fresh. Show me."* |
| **Mid-session drift** (the bookend protocols don't protect this) | Use scenario B interventions early and often. |

---

## Files this system depends on

**Global (apply to every codebase on this machine):**

| Path | Purpose |
|---|---|
| `~/.claude/skills/sosp/SKILL.md` | SOSP skill — pre-flight checklist + auto-trigger description |
| `~/.claude/skills/codegraph-auto-init/SKILL.md` | Codegraph init hook |
| `~/.claude/SESSION_PROTOCOLS.md` | This playbook (always readable by either of us) |

**Project-scoped (per-project, only loads when working in that directory):**

| Path | Purpose |
|---|---|
| `~/.claude/projects/<project-slug>/memory/MEMORY.md` | Project memory index |
| `~/.claude/projects/<project-slug>/memory/project_landmines.md` | Project-specific gotchas SOSP must check |
| `~/.claude/projects/<project-slug>/memory/feedback_*.md` | Project-specific behavioral rules |
| `<PROJECT_ROOT>/working.md` | Cross-session state file written by EOSP |

---

## How to keep this doc honest

If a scenario doesn't match reality, edit this file. Don't let it drift into ritual. The doc is only useful while it reflects how we actually work.

When a new failure mode emerges that any of these phrases would have prevented, add it to the relevant table. When a phrase stops triggering reliably, replace it with one that does.
