---
name: codegraph-auto-init
description: >
  Auto-initializes CodeGraph (semantic code knowledge graph) when working in a codebase
  large enough to benefit from it. Triggers automatically via hook on session start —
  no user intervention needed. Also use this skill when asked to initialize, check,
  or explain CodeGraph status for the current project.
---

# CodeGraph Auto-Init

CodeGraph pre-indexes codebases into a SQLite knowledge graph (symbols, call graphs,
import trees) and exposes it as an MCP server. This eliminates expensive grep/find/Read
exploration — one `codegraph_explore` call returns source sections from all relevant
files instead of fanning out across dozens of tool calls.

## Initialization Criteria (all must be true)

| Criterion | Threshold | Reason |
|-----------|-----------|--------|
| Source file count | ≥ 30 files | Filters throwaway scripts; even ~100-file repos see 77% fewer tool calls (Alamofire benchmark) |
| `.codegraph/` absent | must not exist | Idempotent — skip if already indexed |
| Git repo | must have `.git/` | Ensures it's a real project, not a temp dir |
| CodeGraph binary | must be executable | Skip silently if not installed |

**Supported languages:** TypeScript, JavaScript, Python, Go, Rust, Java, C#, PHP, Ruby,
C, C++, Swift, Kotlin, Dart, Lua, Svelte (and more via codegraph's 19-language support).

## How It Runs Automatically

A `UserPromptSubmit` hook fires `check-and-init.sh` on every session message. The script
exits in milliseconds when criteria aren't met (directory check first). On a qualifying
repo it runs `codegraph init -i` once — subsequent messages find `.codegraph/` and exit
immediately.

## Manual Invocation

If you need to check or force initialization (adjust path to your local install):

```bash
# Check status
codegraph status

# Force init on current project
codegraph init -i

# Re-index after large changes
codegraph index

# Sync incremental changes
codegraph sync
```

## Using CodeGraph Tools (when .codegraph/ exists)

**In the main session — only lightweight lookups:**
- `codegraph_search` — find symbols by name
- `codegraph_callers` / `codegraph_callees` — trace call flow
- `codegraph_impact` — check blast radius before editing
- `codegraph_node` — single symbol details
- `codegraph_status` — index health

**For exploration — always spawn an Explore agent** with this instruction:
> This project has CodeGraph initialized (.codegraph/ exists). Use `codegraph_explore`
> as PRIMARY tool — returns full source sections from all relevant files in one call.
> Do NOT re-read files codegraph_explore already returned. Only fall back to grep/Read
> for files under "Additional relevant files" or if codegraph returned no results.

Never call `codegraph_explore` or `codegraph_context` directly in the main session —
they return large source dumps that fill the main context window.

## When to Re-Init

- After a major refactor changes the shape of the codebase
- After `codegraph uninit` removed the index
- If `codegraph status` reports index is stale or corrupt
