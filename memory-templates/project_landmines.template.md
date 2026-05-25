---
name: project-landmines
description: Project-specific gotchas SOSP must check on every session — fill in for your project
metadata:
  type: project
---

Specific landmines in `<PROJECT_NAME>` that the global `sosp` skill should surface at session start. The skill itself handles generic verification (git topology, deployment URL bundle hashes, entry-point imports, untracked files). This file lists what's **unique to this codebase** so SOSP knows what extra red flags to watch for.

Replace each section below with your project's actual gotchas. Delete sections that don't apply.

---

**Parallel architectures.** (Delete if not applicable.)
> Two engines co-exist in the repo:
> - `<path/to/engine-a.ext>` (`<approach A description>`)
> - `<path/to/engine-b.ext>` (`<approach B description>`)
>
> Only the one imported by `<entry-point-file>` is actually running. Branch names lie — the `<example-branch>` branch's committed code may not reflect its name.

**Locked deployment URL.** (Delete if not applicable.)
> `<your-production-url>` is the production URL bound to a specific production branch in `<your-hosting-platform>`. Deploys from any non-production branch land on `<preview-url-pattern>`, NOT on the production URL. Always curl whatever URL the user is testing on and confirm the bundle hash matches the latest local build.

**Untracked load-bearing files.** (Delete if not applicable.)
> Several files essential to the deployed app are not in git:
> - `<path/to/untracked-file-1>`
> - `<path/to/untracked-file-2>`
>
> These have no git history to revert to. A careless `git checkout` destroys them. Always run `git status -uall <critical-dir>/` before any branch operation.

**Deploy command.** (Replace with your actual pipeline.)
> Build + deploy is non-negotiable after every change:
> ```bash
> <build-command> && <deploy-command>
> ```
> Run from `<PROJECT_ROOT>`.

---

Add as many landmine sections as you need. Each one should be a concrete, project-specific trap that has burned a previous session or that a new contributor would not discover from reading the code alone.

See also: [[feedback_always_deploy]], [[feedback_no_temporal_claims]].
