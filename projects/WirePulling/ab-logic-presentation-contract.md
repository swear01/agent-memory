---
title: WirePulling AB logic presentation changes preserve the evaluator contract
scope: projects/WirePulling
project: WirePulling
tool: Unity
status: active
confidence: high
evidence: >-
  Presentation-layer regeneration completed and the targeted Unity EditMode
  suite reported 42/42 passed, including truth-table, missing-input, and
  terminal-compatibility coverage.
created: 2026-08-18
updated: 2026-08-18
tags:
  - unity
  - logic-gates
  - truth-table
  - presentation
source_refs:
  - codex-b001-wirepulling:019e3703-4c4f-7802-b082-a6cac6c7a8d7
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Contract

For the WirePulling AB logic puzzle, a gate-visual redesign must preserve:

- `Bridge = A XOR B`
- `Lock = A AND B`
- no added XOR gate
- existing evaluator, wire compatibility, and truth-table completion semantics

The scene builder may change generated gate panels, sockets, labels, and explanatory text while keeping the evaluator contract unchanged. The targeted EditMode suite passed 42/42 after regeneration.

This validates logic and static generation only. Unity MCP reported no registered Editor instance, so Play Mode, runtime Console, and final visual behavior still require fresh verification.
