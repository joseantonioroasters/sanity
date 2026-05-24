# sanity — Session Continuity Protocols for Claude Code

A three-tool system for reliable multi-session work on existing codebases with [Claude Code](https://claude.com/claude-code) and similar agentic coding assistants.

## What's in this repo

| Path | Purpose |
|---|---|
| `SESSION_PROTOCOLS.md` | The shared playbook — what to say in each scenario, what the assistant does in response |
| `skills/verify-latest-source/SKILL.md` | Start-of-Session Protocol (SOSP) skill — pre-flight verification of source-of-truth before any edit |
| `skills/codegraph-auto-init/SKILL.md` | Auto-initializes [CodeGraph](https://github.com/davidsuper/codegraph) when working in qualifying repos |
| `memory-templates/` | Templated feedback rules to drop into project-scoped memory |

## The three tools

| Tool | Role | Trigger |
|---|---|---|
| **EOSP** (end-of-session protocol) | Writes dense state snapshot into memory at session close | User types `EOSP` |
| **SOSP** (start-of-session protocol) | Verifies git topology, deployment URL, active imports, untracked files, deployed bundle before acting | Auto-fires on coding requests in existing codebases |
| **CodeGraph** | Persistent SQL index of symbols, imports, call graph | Auto-init via UserPromptSubmit hook in qualifying repos |

Together: EOSP writes the story → CodeGraph indexes the structure → SOSP cross-checks both against reality before any work begins.

**Scope:** modifications on existing codebases. Greenfield projects don't benefit (nothing to verify).

## Installation

1. **Copy skills into your Claude skills directory:**

   ```bash
   cp -r skills/verify-latest-source ~/.claude/skills/
   cp -r skills/codegraph-auto-init ~/.claude/skills/
   ```

2. **Place the playbook where you can reference it:**

   ```bash
   cp SESSION_PROTOCOLS.md ~/.claude/
   ```

3. **For each project that benefits, add the project-scoped memory file** that enforces SOSP invocation. The path depends on your Claude memory layout — typically something like `~/.claude/projects/<project-slug>/memory/`. Adapt `memory-templates/feedback_invoke_verify_at_start.template.md` and drop it in.

4. **Add the other feedback memories** as relevant to your project:
   - `feedback_always_deploy.template.md` — for projects with a build+deploy step
   - `feedback_no_temporal_claims.template.md` — universal; safe to add to any project
   - `feedback_invoke_verify_at_start.template.md` — for projects where SOSP must fire reliably

5. **EOSP skill** is currently distributed as part of Anthropic's skills library. If your environment has it, the trigger phrase is `EOSP`. If not, you can write your own EOSP-equivalent skill that writes structured state into your memory layout.

## Why this exists

Multi-session agentic work fails in predictable ways:

- The assistant edits a stale file in a worktree while newer commits live elsewhere
- It deploys to a preview URL while the user tests production
- It reverts to an "old working state" that's internally inconsistent
- It treats branch names or filenames as proof of architecture
- It loses functionality by `git checkout`ing past untracked load-bearing files
- It blames browser cache without verifying which bundle the URL actually serves
- It makes confident temporal claims about code state when it has no real timeline anchor

This system mitigates each of these via:
- **Reliable verification** (SOSP) before any action
- **Faster structural queries** (CodeGraph) so verification is cheap enough to be a default
- **Cross-session memory** (EOSP) so context survives between conversations

## Read the playbook

Open [`SESSION_PROTOCOLS.md`](./SESSION_PROTOCOLS.md) for the working agreement: what you say, what the assistant does, what to do mid-session when things drift.

## License

MIT — adapt freely for your own workflow.
