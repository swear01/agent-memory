---
title: Permission-question wire formats must not turn skips into consent
scope: tools/hapi
project: hapi
tool: agent protocol adapters
status: needs-review
confidence: medium
tags:
  - hapi
  - permissions
  - acp
  - protocol
  - safety
source_refs:
  - agent-memory:issue-15
created: 2026-08-20
updated: 2026-11-28
generated_by: openai-codex/gpt-5.6-luna
---

# Question boundary

Permission and question protocols are flavor-specific. Claude Code answers
may be keyed by question text, Codex request-user-input uses nested stable
identifiers, and Cursor ACP uses stable question and option identifiers with
only a limited label fallback. Do not translate these shapes through one
untyped generic answer map.

A headless or legacy adapter that emits a synthetic `skipped` result has not
received operator consent. It must be surfaced as no-answer, blocked, or an
explicitly non-consenting outcome. Only a narrowly identified compatibility
marker may be intercepted; real legacy traffic must remain visible.

A historical headless `cursor-agent` case returned `questions skipped` with
`is_error=false` where no question UI was available. Require evidence from the
operator or a verified interactive UI before treating any response as consent.
The issue's closed state only establishes its lifecycle.
[Historical case](../../archive/tools/hapi/synthetic-question-skip-is-not-consent.txt)

This note remains `needs-review` until the released adapters and their
operator-approval paths are verified together.
