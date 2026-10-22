---
title: HAPI 的 Claude 背景任務計數原本永遠是 0；改用 background_tasks_changed snapshot（PR #2102）
scope: projects/hapi-claude-background-tasks
project: hapi
status: active
confidence: high
evidence: 2026-10-22 線上 hub 1188 個 session 的 backgroundTaskCount 全部是 0；用 eb0538a4 session 的真實訊息跑舊的 extractBackgroundTaskDelta，開始和結束都回傳 null；當天 CLI debug log 有 background_tasks_changed，包含空清單。PR tiann/hapi#2102 CI 全綠、HAPI Bot 在 head 647d68ede 沒有新問題，等維護者 merge。
created: 2026-10-22
updated: 2026-10-22
tags:
  - hapi
  - claude
  - session-list
---

- Claude Code 2.1.302 的 SDK／remote 模式，用 `system/background_tasks_changed` 回報背景任務，內容是完整的 `tasks` 清單，最後一個任務結束時會送空清單。`task_started`／`task_notification` 也存在，但前景 Bash 也會觸發。stream 裡不會出現 `<task-notification>` 這筆 user record，它在 JSONL 裡是內部的 queued_command attachment。「Command running in background」的 tool_result 是包在 `type:'user'` record 裡送來的。
- CLI 的 `OutgoingMessageQueue` 只轉送 `isClaudeChatVisibleMessage` 判定為可見的 record。不在聊天裡顯示的 system subtype 不會到 hub，除非另外放行。所以在假設 hub 收得到某個 SDK 事件之前，要先查這個 filter。完整路徑：SDK 事件 → sdkToLogConverter → OutgoingMessageQueue → hub sessionHandlers。
- 左欄的相對時間用的是 `updatedAt`。上游 #1249 之後只有使用者 prompt 才會更新它，所以「20m ago」是距離上次 prompt 的時間，不是 agent 最後一次輸出的時間。
- PR #2102：hub 用 snapshot 直接設定 `backgroundTaskCount`，當成 session 狀態套用，不存成訊息；session 列表上，等待背景任務時顯示藍色慢速旋轉圖示。local（PTY）模式的 JSONL 沒有對應的 snapshot，所以這部分不涵蓋。
