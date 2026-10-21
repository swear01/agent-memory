---
title: "Claude Code 2.x subagent transcript 佈局與 HAPI local mode 缺口"
scope: tools/claude-code
tags: [claude-code, subagent, sidechain, hapi, parentToolUseId]
status: active
updated: 2026-10-21
---

# Claude Code 2.x subagent transcript 佈局

Claude Code 2.x（實測 2.1.302）不再把 subagent（Agent/Task sidechain）寫進主 session log：

- 主 log：`<claude-projects>/<dir>/<sessionId>.jsonl`，只有 `Agent` tool_use / tool_result，**沒有任何 `isSidechain:true` 行**。
- subagent：`<claude-projects>/<dir>/<sessionId>/subagents/agent-<agentId>.jsonl`，行上有 `isSidechain:true`、`agentId`，但**沒有 parent tool_use id**；root 是 `type:user`、`parentUuid:null`、content 為 prompt 字串。
- 同名 `agent-<agentId>.meta.json`：`{"agentType", "description", "toolUseId", "spawnDepth", "requestShape": "background"|...}`，`toolUseId` 就是母 `Agent` tool_use id（`toolu_…`）。spawn 時就建立，之後會被整檔改寫。
- subagent 檔內只有 `user`/`assistant`/`attachment` 與 `api-request*` 類型，沒有 `summary` / `ai-title`。

SDK / `claude -p --output-format stream-json --verbose` 路徑則不同：前景與背景 subagent 的 assistant/user 訊息都帶 `parent_tool_use_id`；背景 subagent 沒有 prompt root，但有 `system task_started {task_id, tool_use_id}`。

## HAPI 影響

- Remote（SDK）模式：`sdkToLogConverter` 自 #1175（v0.24.0）起轉成 `parentToolUseId`，web `tracer.ts` 以此分組 → 正常。#1904 回報者用 0.23.4，才會 hub 裡 0 筆 `parentToolUseId`。
- Local 模式：`cli/src/claude/utils/sessionScanner.ts` 原本只讀主 log → subagent 活動完全沒送到 hub。修法（PR tiann/hapi#2091）：掃 `subagents/agent-*.jsonl`，從 meta.json 補 `parentToolUseId`，啟動時既有 subagent 檔視為已處理。
- 驗證技巧：用 `claude -p --model haiku` 跑一次前景、一次背景 Agent 產生真實 transcript，再把 scanner 輸出餵 web `normalizeDecryptedMessage → traceMessages → reduceChatBlocks` 看是否掛在 Agent card 下。

相關：`tools/hapi/subagent-executed-model-evidence.md`。
