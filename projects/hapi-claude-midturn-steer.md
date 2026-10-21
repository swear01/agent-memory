---
title: Claude Code stream-json 現在支援 mid-turn steer，HAPI 仍把 Claude 排除在 Steer 之外
scope: projects/hapi-claude-midturn-steer
project: hapi
status: active
confidence: high
evidence: 2026-10-21 用 claude 2.1.302 (--input-format stream-json, haiku) 實測；第一個 tool_use 後注入 user message，turn 在下一個 tool boundary 轉向、跳過後續步驟、只產生一個 result。
created: 2026-10-21
updated: 2026-10-21
tags:
  - hapi
  - claude
  - steer
  - stream-json
---

- tiann/hapi #888 的結論「stream-json mid-turn stdin 會被忽略」是在 claude 2.1.175 驗證的，已過時。
- 2.1.302：turn 進行中寫進 stdin 的 user message，會在下一個 tool boundary 被吃進「同一個 turn」（不會另開 turn、不會多一個 result，stream 也不會 echo 這則 user message）。
- HAPI 沒有 Claude Steer 的原因：`shared/src/modes.ts` 的 `STEERING_SUPPORTED_FLAVORS` = codex/cursor/pi；`cli/src/claude/claudeRemote.ts` 只在 `result` 之後才 `scheduleNextMessage()`，所以 web 送的訊息會一直留在 MessageQueue2，等到 turn 結束。
- 要實作的話：把 claude 加進 gate，並在 claude 的 CLI 處理 `SteerQueuedMessage` RPC，turn 進行中直接 push 到 `messages` iterable（參考 codex/cursor 的 handler）。
