---
title: Upstream HAPI issue triage standard ("worth handling")
scope: tools/hapi/upstream-issue-triage-standard
project: hapi
status: active
updated: 2026-10-27
tags: [hapi, github, issue, triage]
redaction: passed
---

When asked which tiann/hapi issues are worth handling, use the 2026-07-13 standard:

- Filter: no assignee, no open PR and no PR merged since the issue was filed (search open and merged PR bodies by keyword, not only `#N` links; #2062 stayed open after merged #2066 covered it), not claimed in comments, gap still present on current `upstream/main` (verify in code).
- Report: date + open counts + main SHA; ranked table (issue, value/root cause, S/M/L); quick small ones; "looks open but don't pick" (has PR, already fixed, fork-only code, unreproduced); one pick per goal.
- Issues citing fork-only code (e.g. `peerSpawnDefaults`, `MachineTagConflictError`) are not actionable upstream.
