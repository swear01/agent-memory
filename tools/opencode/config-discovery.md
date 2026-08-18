---
title: Bound agent-harness config discovery to known directories
scope: tools/opencode
tool: OpenCode/Zed
status: active
confidence: high
evidence: >-
  Unbounded home-directory searches were aborted or dominated by node_modules;
  targeted searches located the relevant Zed and OpenCode configuration roots.
created: 2026-08-18
updated: 2026-08-18
tags:
  - opencode
  - zed
  - configuration
  - diagnostics
source_refs:
  - opencode-b001:ses_1d3a627aeffe0E1JRN03lMitVT
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Discovery rule

When debugging an agent-harness configuration, do not start with an unbounded recursive glob from `$HOME`. It can be aborted or overwhelmed by package directories such as `node_modules`.

Search known roots directly, such as `$HOME/.config/zed` and `$HOME/.config/opencode`, then inspect the specific settings/config files and the active launch environment. Treat a targeted negative result as evidence only for the paths that were actually checked.
