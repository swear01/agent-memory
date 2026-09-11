---
title: Upstream HAPI PR triage closures for superseded behavior
scope: tools/hapi/upstream-pr-triage-closures
project: hapi
status: active
updated: 2026-09-11
tags:
  - hapi
  - github
  - pull-request
  - triage
source_refs:
  - github:tiann/hapi#1421
  - github:tiann/hapi#1451
  - github:tiann/hapi#1610
redaction: passed
---

On 2026-09-11, the upstream HAPI PR triage was verified against live GitHub state using the local `gh` CLI.

- `tiann/hapi#1421` was closed because current `main` already persists and restores the covered New Session launch settings, so the old change is superseded.
- `tiann/hapi#1451` was closed because current `main` New Session uses runner-reported Agent availability and lists only Agents that are currently available/installed; a separate visibility preference is no longer needed.
- `tiann/hapi#1610` was already closed because the current mainline flow includes the relevant Codex/Cursor queued-message steering behavior.

For future old-PR triage, re-check the current `main` behavior before deciding whether a PR still has product value; close superseded PRs with a short rationale and verify the resulting GitHub state with `gh pr view`.
