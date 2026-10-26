---
title: HAPI 的 Claude 背景任務計數原本永遠是 0；改用 background_tasks_changed snapshot（PR #2102）
scope: projects/hapi-claude-background-tasks
project: hapi
status: active
confidence: high
evidence: 2026-10-22 線上 hub 1188 個 session 的 backgroundTaskCount 全部是 0；用 eb0538a4 session 的真實訊息跑舊的 extractBackgroundTaskDelta，開始和結束都回傳 null；當天 CLI debug log 有 background_tasks_changed，包含空清單。PR tiann/hapi#2102 CI 全綠、HAPI Bot 在 head 647d68ede 沒有新問題。2026-10-25 依使用者回饋改成共用 SessionSpinner 並 merge upstream/main，新 head a455fea04，web typecheck 與相關 vitest 通過，等維護者 merge。#2102 已於 2026-10-25 merge。2026-10-26 後續 PR tiann/hapi#2177：HAPI Bot 在 head dc1b50499 沒有問題，CI 全綠（含 iOS／Android），等維護者 merge。
created: 2026-10-22
updated: 2026-10-26
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
- 後續 PR #2177（2026-10-26）：狀態面板原本顯示「exact command unavailable」，因為 hub 只存數量，web 只能從已載入的聊天紀錄猜指令。改成把 snapshot 的 `tasks`（`{id, type, description}`）存成 `Session.backgroundTasks`，並透過 SSE patch 送出，數量從清單長度推導。snapshot 本身沒有指令；`local_bash` 的 `description` 是 Bash 工具的 `description` 參數。web 只從「啟動訊息」比對任務 ID 來補上完整指令：「Command running in background with ID: X」和自動轉背景的「moved to the background (ID: X)」兩種措辭都要支援，舊 regex 只認前者。不能只要結果含有 ID 就算，例如 `ls` 列出 task output 檔名也會含 ID，這樣會被誤認成任務的指令（HAPI Bot 抓到）；比對不到時退回 description。
- 原生端的 `SessionPatch` decoder 是 strict 的，對應 zod `.strict()`：Android 是 `SessionPatches.KNOWN_KEYS`，iOS 是 `CodingKeys` 加 `rejectUnknownKeys`。hub 的 session patch 只要新增 key，就要同時加進這兩處，否則原生 client 會拒收整個 patch，改走 REST 重新抓。完整 `Session` 的 decoding 才是寬鬆、會忽略未知欄位的，不要因為 full-session 寬鬆就以為 patch 也一樣（#2177 一度誤判，HAPI Bot 抓到）。
