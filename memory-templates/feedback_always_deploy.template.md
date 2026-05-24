---
name: feedback-always-deploy
description: Always deploy after every code change in this project — never wait to be asked
metadata:
  type: feedback
---

Always run build + deploy as the final step of every code change in <PROJECT_NAME>. Do not wait for the user to ask.

**Why:** Rapid change/test workflow — every change needs to be live immediately for testing on the target environment (e.g. mobile devices, separate test machines).

**How to apply:** After every edit to source files that affect the deployed app, run the project's full build + deploy chain. Replace the command below with your project's exact deploy pipeline:

```bash
# Example — adapt to your project:
<build-command> && <deploy-command>
```

This is non-negotiable for this project. If a deploy fails, surface the error; don't silently skip the step.
