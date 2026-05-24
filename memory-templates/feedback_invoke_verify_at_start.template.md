---
name: feedback-invoke-verify-at-start
description: At the start of any coding task in this project, invoke the verify-latest-source skill before reading or editing any file
metadata:
  type: feedback
---

At the start of any coding task in <PROJECT_NAME> at `<PROJECT_ROOT>`, invoke the `verify-latest-source` skill before reading or editing any file. Non-negotiable.

**Why:**
- [List the specific landmines that make verification critical for this project. Examples below — adapt to your reality.]
- Multiple parallel architectures co-exist in this repo: <e.g. engine A in src/x.js, engine B in src/y.js>. Only the one imported by the entry point is actually running. Branch names lie.
- Production URL `<your-production-url>` is locked to a specific branch in the deployment system. Deploys from other branches land on preview URLs. Confusing the two has consumed entire sessions.
- Several load-bearing files are untracked in git (`<list any critical untracked files>`). They have no git recovery point and a careless `git checkout` destroys them.
- Past sessions reverted files to "the working state" based on commit hashes without checking that the snapshot was internally consistent — creating new regressions while purporting to fix old ones.

**How to apply:**
Before the first file read or edit in any new session that touches `<PROJECT_ROOT>`, walk the `verify-latest-source` checklist. Specifically confirm:
1. Which deployment URL serves which build (curl bundle hash)
2. What the entry-point file actually imports (active architecture, ignore branch name)
3. Which imported files are untracked (flag for the user)
4. Whether the user's reported URL matches the latest deploy or a stale one

Then read the project's `working.md` state file (if it exists) before proposing any plan.

See also: [[feedback_no_temporal_claims]], [[feedback_always_deploy]].
