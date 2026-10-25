---
title: HAPI 的 Claude 背景任務計數原本永遠是 0；改用 background_tasks_changed snapshot（PR #2102）
scope: projects/hapi-claude-background-tasks
project: hapi
status: active
confidence: high
evidence: 2026-10-22 線上 hub 1188 個 session 的 backgroundTaskCount 全部是 0；用 eb0538a4 session 的真實訊息跑舊的 extractBackgroundTaskDelta，開始和結束都回傳 null；當天 CLI debug log 有 background_tasks_changed，包含空清單。PR tiann/hapi#2102 CI 全綠、HAPI Bot 在 head 647d68ede 沒有新問題。2026-10-25 依使用者回饋改成共用 SessionSpinner 並 merge upstream/main，新 head a455fea04，web typecheck 與相關 vitest 通過，等維護者 merge。
created: 2026-10-22
updated: 2026-10-25
tags:
  - hapi
  - claude
  - session-list
---

- Claude Code 2.1.302 的 SDK／remote 模式，用 `system/background_tasks_changed` 回報背景任務，內容是完整的 `tasks` 清單，最後一個任務結束時會送空清單。`task_started`／`task_notification` 也存在，但前景 Bash 也會觸發。stream 裡不會出現 `<task-notification>` 這筆 user record，它在 JSONL 裡是內部的 queued_command attachment。「Command running in background」的 tool_result 是包在 `type:'user'` record 裡送來的。
- CLI 的 `OutgoingMessageQueue` 只轉送 `isClaudeChatVisibleMessage` 判定為可見的 record。不在聊天裡顯示的 system subtype 不會到 hub，除非另外放行。所以在假設 hub 收得到某個 SDK 事件之前，要先查這個 filter。完整路徑：SDK 事件 → sdkToLogConverter → OutgoingMessageQueue → hub sessionHandlers。
- 左欄的相對時間用的是 `updatedAt`。上游 #1249 之後只有使用者 prompt 才會更新它，所以「20m ago」是距離上次 prompt 的時間，不是 agent 最後一次輸出的時間。
- PR #2102：hub 用 snapshot 直接設定 `backgroundTaskCount`，當成 session 狀態套用，不存成訊息；session 列表上，等待背景任務時顯示藍色慢速旋轉圖示。local（PTY）模式的 JSONL 沒有對應的 snapshot，所以這部分不涵蓋。
- 使用者回饋（2026-10-25）：藍色（背景任務）跟綠色（turn 進行中）是同一種指示，只差顏色，所以要走同一段程式碼。#2102 第一版的藍色有自己的 class 常數、外層 `<span>` 和「Running」文字，手機上看起來跟綠色不一樣，被使用者退回。改法是新增 `SessionSpinner`（`web/src/components/SessionAttentionIndicator.tsx`），兩種狀態都用它，`background` 時只換成 `text-blue-400`，並加測試比對兩個 SVG 除了顏色以外完全相同。以後幫既有指示加顏色變體時，直接重用原本的元件，只參數化顏色，不要另寫一份 markup 或加標籤。
