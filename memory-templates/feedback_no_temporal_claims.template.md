---
name: feedback-no-temporal-claims
description: Do not make temporal claims about codebase state ("today", "recent", "last working", "before this session") — no real timeline awareness exists
metadata:
  type: feedback
---

Never make temporal claims about when code was in a particular state. Avoid phrases like "today's session," "before today," "the recent working state," "last commit before this session," or "what was working long ago" unless the user has explicitly anchored those terms in this conversation.

**Why:** Git commit hashes are labeled snapshots, not points on a timeline that can be navigated. There is no real awareness of:
- When commits were authored in wall-clock time relative to "now"
- Which snapshots correspond to the user's mental model of "working" or "recent"
- Whether values seen in old snapshots (e.g. default config names, feature flags) were actually live in deployment at that moment
- What the user remembers as the last good state

Confidently asserting "X was the state before today" when X is just whatever the file contained at commit Y produces wrong, confusing recoveries.

**How to apply:**
- When proposing reverts or recoveries, describe snapshots by what they CONTAIN, not when they were live: "commit XYZ has feature X in the entry point" — not "the state before today."
- Ask the user to identify the snapshot they trust rather than guessing.
- If asked about "the last working version," show `git log --oneline` and let the user point.
- Treat phrases like "today," "this session," "recently" as anchored only to messages explicitly in the current conversation — never to git history or memory snapshots.

This rule is universal (safe to drop into any project's memory).
