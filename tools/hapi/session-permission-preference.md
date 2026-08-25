---
title: New HAPI sessions default to YOLO permission mode
scope: tools/hapi
project: hapi
tool: session creation
status: active
confidence: high
tags:
  - hapi
  - sessions
  - permissions
  - user-preference
source_refs:
  - user-confirmed:2026-08-26
created: 2026-08-26
updated: 2026-08-26
generated_by: openai-codex/gpt-5.6-sol
---

# Session permission preference

When creating a new HAPI coding-agent session for this operator, use YOLO mode
unless the operator explicitly selects another permission mode. YOLO changes
approval handling only; it does not expand the requested task scope or
authorize destructive actions.

After creation, verify the recorded session permission mode together with the
agent, model, machine, workspace, and ready state.
