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

HAPI Web 的 effort 動態回報屬另一條 ACP adapter，見 [OpenCode effort 辨識與回讀](../hapi/opencode-dynamic-effort-reconciliation.md)。不以任一項操作已完成推定 Zed history 可恢復。

[原始調查](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/opencode/zed-acp-routing.md)
