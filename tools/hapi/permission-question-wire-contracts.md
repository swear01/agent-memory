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
updated: 2026-08-20
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

This note remains `needs-review` until the released adapters and their
operator-approval paths are verified together.
