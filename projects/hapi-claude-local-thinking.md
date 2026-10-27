---
title: HAPI local-mode Claude 的 thinking 由 hooks + transcript 中斷標記推導（PR #2209）
scope: projects/hapi-claude-local-thinking
project: hapi
status: active
confidence: high
evidence: 2026-10-27 upstream main 85ccab1ac；PR tiann/hapi#2209 head 76704af7d HAPI Bot 無發現、CI 全綠，等維護者 merge。Claude Code 2.1.302 binary 內含 StopFailure hook 與 "[Request interrupted by user" 字串。
created: 2026-10-27
updated: 2026-10-27
tags:
  - hapi
  - claude
  - hooks
---

# 問題（issue #1864）

Remote Claude 由 SDK stream 切換 `thinking`；local（TUI）路徑沒有 stream，也從不呼叫 `onThinkingChange`，local hook 設定也沒註冊回合結束 hook，所以 web 一直顯示 idle。

# 修法

- `generateHookSettings` 的 local（`includeLocalPermissions`）設定加 `Stop`、`StopFailure`。
- `runClaude` 的 `onSessionHook`：`UserPromptSubmit`/`PreToolUse` → true，`Stop`/`StopFailure`/`SessionEnd` → false（沿用 subagent、local mode、current session guard）。
- Esc 中斷不觸發 `Stop`；Claude Code 只在 transcript 寫入 user 訊息 `[Request interrupted by user]` / `[Request interrupted by user for tool use]`。`claudeLocalLauncher` 以此清除 thinking，離開 local mode 時一律重設。

# 坑

Transcript scanner（fs watch + 3 秒 polling）與 hook HTTP 是兩條獨立管道：中斷後立刻送新 prompt，舊的中斷標記可能晚於新回合的 `UserPromptSubmit` 抵達而誤清 thinking。解法：記錄 `UserPromptSubmit` 的時間（`session.localTurnStartedAt`），略過 transcript `timestamp` 較舊的標記。HAPI Bot 首輪即指出這個 race。
