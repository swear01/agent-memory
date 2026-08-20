---
title: Zed OpenCode ACP routing needs runtime verification
scope: tools/opencode
tool: OpenCode/Zed
status: needs-review
confidence: medium
evidence: >-
  Zed used a registry-mode OpenCode agent entry while the local investigation
  changed it to an explicit `opencode acp` command; restart, ACP logs, and
  session recovery were not tested.
created: 2026-08-18
updated: 2026-08-18
tags:
  - opencode
  - zed
  - acp
  - configuration
  - needs-review
source_refs:
  - opencode-b001:ses_1d3a627aeffe0E1JRN03lMitVT
  - hapi:cli/src/opencode/opencodeRemoteLauncher.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Configuration boundary

Zed's registry-mode OpenCode entry and an explicit command-mode entry are different integration paths. When ACP sessions cannot discover OpenCode history, inspect the active Zed agent-server configuration and consider an explicit `opencode acp` command with a machine-resolved executable.

Do not treat an edit as a verified fix. Confirm the Zed restart, ACP logs, active working directory, session listing, and actual history recovery before marking the change active.

# Runtime option boundary

When an OpenCode ACP build exposes runtime effort options, derive the
available values from ACP rather than hardcoding a preset list. If
`session/set_config_option` is unavailable, disable inline switching
explicitly. Unsupported requested values must be rejected or coerced to a
supported value, and a failed update must roll back the HAPI/web state. This
remains `needs-review` until restart and history recovery are verified on the
released build.
