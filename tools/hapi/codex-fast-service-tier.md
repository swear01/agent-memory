---
title: Codex Fast is an explicit service tier, not reasoning effort
scope: tools/hapi
project: hapi
tool: Codex app-server
status: needs-review
confidence: medium
tags:
  - hapi
  - codex
  - service-tier
  - model-catalog
source_refs:
  - hapi:cli/src/modules/common/codexModels.ts
  - hapi:shared/src/apiTypes.ts
  - hapi:hub/src/sync/sessionModel.test.ts
  - agent-memory:issue-15
created: 2026-08-20
updated: 2026-08-20
generated_by: openai-codex/gpt-5.6-luna
---

# Service-tier boundary

Fast is a service tier, separate from reasoning effort. Availability is
catalog- and model-dependent, and may also depend on authentication. Stored
`fast`, explicit `standard`, and an unset value are distinct states; do not
collapse them into one boolean default.

At the wire boundary, the stored Fast choice maps to the app-server request
priority expected by the Codex model catalog. Unsupported service-tier values
must be rejected rather than silently rewritten. A model picker should expose
Fast only when the live catalog and current authentication support it.

This note remains `needs-review` until the released fleet build and the
provider-dependent availability path are verified together.
