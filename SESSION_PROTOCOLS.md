# Session Protocols — Working Agreement

A shared playbook between a user and an agentic coding assistant using the three-tool system:
**EOSP** (end-of-session memory write) + **SOSP** (start-of-session verification) + **CodeGraph** (persistent code index).

Scoped to **modifications on existing codebases**, not greenfield projects.

---

## The three tools at a glance

| Tool | Trigger | What it does | Direction |
|---|---|---|---|
| **EOSP** (skill) | User types exactly `EOSP` | Writes a dense state snapshot into project memory files | Writes |
| **SOSP** (`verify-latest-source` skill + project memory rule) | Auto-fires on most coding requests in existing codebases | Verifies git topology, deployment URL, active imports, untracked files, deployed bundle | Reads + verifies |
| **CodeGraph** (`codegraph-auto-init` skill) | Auto-init via UserPromptSubmit hook in qualifying repos | Builds persistent SQL index of symbols, imports, call graph | Indexes |

Together: EOSP writes the story → CodeGraph indexes the structure → SOSP cross-checks both against reality before the assistant acts.

---

## Scenarios — what you say, what the assistant does

### A. Start of a new session on an existing codebase (the main case)

**You say** (any of these is enough):
- Just state your goal: *"Fix the camera permission on Android"* / *"Change the loader color to blue"* / *"Deploy the latest"*
- Or explicitly: *"Run SOSP first"* / *"Verify source of truth before starting"*

The skill description should auto-fire on words like *fix, change, add, deploy, revert, what's wrong*. The explicit form guarantees it.

**You provide upfront (saves several turns):**
- The deployment URL you're testing on right now
- One sentence on the last behavior you remember working
- Branch preference if you have one

**The assistant does:**
1. Reads project memory (`MEMORY.md` + the files it links)
2. Reads `working.md` if it exists at the project root
3. Runs `verify-latest-source` checklist: git topology → deployment URL bundle hashes → entry-point imports → untracked critical files
4. Queries CodeGraph (if initialized) for active import graph instead of grepping
5. Reports a one-screen situation summary and a plan
6. Waits for confirmation OR proceeds if the plan is clearly tactical

---

### B. Mid-session — you sense drift

Don't wait until things get worse. These short phrases re-anchor the assistant:

| You say | The assistant does |
|---|---|
| *"Verify what's deployed"* | Re-curls the URL, greps bundle markers, compares to local build |
| *"Re-check entry-point imports"* | Re-greps imports, states active architecture |
| *"What URL is your deploy landing on?"* | States explicitly: production vs preview alias vs unique hash URL |
| *"Stop. Re-verify."* | Halts, re-runs full SOSP checklist before continuing |
| *"You're drifting. Show me what you actually know."* | Lists current assumptions with evidence; flags anything being guessed |

Cost of calling these is low. Cost of letting drift accumulate is high.

---

### C. Ending a session cleanly

**You say:** `EOSP` (exact phrase, anywhere in a message)

**The assistant does:** invokes the EOSP skill, which writes a structured snapshot into project memory:
- Current branch + uncommitted state
- What was changed this session and why
- What was deployed (URL + bundle hash)
- What's pending / what to try next
- Gotchas to remember

Then stops and reports the file paths written.

---

### D. Switching to a different codebase

**You say:** [open a new conversation in a different working directory]

**Automatic:** project-scoped memory path changes based on cwd. CodeGraph re-checks for `.codegraph/` in the new repo.

**You should still:** name the project in your first message so the right memory is read.

---

### E. New / greenfield project

**You say:** *"Starting a new project"* or describe greenfield work explicitly.

**The assistant does:** skips SOSP entirely (nothing to verify). CodeGraph won't init until source file count crosses the threshold. EOSP is still useful for tracking decisions made.

---

## Quick-reference card

| Intent | Phrase | What fires |
|---|---|---|
| Start work on existing codebase | "fix / change / add / deploy / revert ..." | SOSP auto-fire |
| Force full verification | "Run SOSP" / "Verify before continuing" | SOSP explicit |
| Re-check deployment | "What's at \<URL\>?" | Curl + bundle marker grep |
| Re-check architecture | "What does the entry-point actually import?" | grep entry-point or CodeGraph |
| Re-check git source | "What's the git topology?" | git worktree + branch + log |
| Stop drift | "Stop. Re-verify." | Halt + re-run SOSP |
| Close session | "EOSP" | EOSP writes snapshot |
| Resume next session | (anything in scenario A) | SOSP reads what EOSP wrote |

---

## What you do NOT need to say (enforced via feedback memory)

These should fire automatically once the relevant memory files are installed (see `memory-templates/`):

- **Always deploy after every change** — assistant builds + deploys without being asked
- **No temporal claims about code** — assistant describes snapshots by content, not by "today" / "recent" / "last working"
- **Project-specific conventions** (file formats, naming) — add as project memory

If the assistant violates any of these, call it out — it means a memory rule isn't firing.

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
| **CodeGraph stale** (~1 sec watcher lag after edits) | If the assistant just edited then queries, results may lag. Always trust reality (git, curl) when CodeGraph and reality disagree. |
| **Memory drift** (EOSP wrote wrong state) | Override directly: *"Memory is wrong about X. Here's the truth: ..."* — the assistant should update the relevant file. |
| **Skill trigger missed** (SOSP didn't auto-fire on subtle phrasing) | Use explicit *"Run SOSP"* instead of relying on heuristic match. |
| **Three layers compound** (CodeGraph stale + memory wrong + assistant trusts majority) | Force re-verify against reality: *"Curl the URL. Read the entry-point fresh. Show me."* |
| **Mid-session drift** (the bookend protocols don't protect this) | Use scenario B interventions early and often. |

---

## Files the system depends on

| Path | Purpose |
|---|---|
| `~/.claude/skills/verify-latest-source/SKILL.md` | SOSP skill definition + checklist |
| `~/.claude/skills/codegraph-auto-init/SKILL.md` | CodeGraph init hook |
| `~/.claude/projects/<project-slug>/memory/MEMORY.md` | Project memory index |
| `~/.claude/projects/<project-slug>/memory/feedback_*.md` | Behavioral rules the assistant follows without being asked |
| `<PROJECT_ROOT>/working.md` | Cross-session state file for the project |
| `~/.claude/SESSION_PROTOCOLS.md` | This playbook |

---

## How to keep this doc honest

If a scenario doesn't match reality, edit this file. Don't let it drift into ritual. The doc is only useful while it reflects how you actually work.

When a new failure mode emerges that any of these phrases would have prevented, add it to the relevant table. When a phrase stops triggering reliably, replace it with one that does.
