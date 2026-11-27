---
title: HAPI 的 Claude 背景任務計數原本永遠是 0；改用 background_tasks_changed snapshot（PR #2102）
scope: projects/hapi-claude-background-tasks
project: hapi
status: active
confidence: high
evidence: Claude Code 2.1.302 SDK、CLI debug 及實際計數觀察；2026-11-28 GitHub 讀回 #2102 和 #2177 已合併。
created: 2026-10-22
updated: 2026-11-28
tags:
  - hapi
  - claude
  - session-list
---

- Claude remote 模式背景計數一直 0 時先核 `system/background_tasks_changed`。這是完整 tasks snapshot，最後一個任務結束會送空清單。`task_started`／`task_notification` 也可能屬前景 Bash。local PTY 的 JSONL 缺少對應 snapshot。
- SDK → `sdkToLogConverter` → `OutgoingMessageQueue` → hub session handlers。Queue 一般只轉發 `isClaudeChatVisibleMessage`，背景 system subtype 要特別放行。Hub 更新 session 狀態並透過 SSE 傳 `{id,type,description}`，計數從清單推導。相對時間的 `updatedAt` 自 #1249 取最後使用者 prompt。
- 只按任務 ID 匹配真正開啟背景的訊息「Command running in background with ID: X」或「moved to the background (ID: X)」，找不到退回 description。普通 `ls` 輸出亦可能帶 ID。Snapshot description 本身不保證完整指令。
- 綠色 turn 與藍色背景都用 `SessionSpinner`，僅參數化顏色，覆蓋 SVG 結構及 mobile。
- 新 SSE `SessionPatch` key 必須更新 Android `SessionPatches.KNOWN_KEYS` 和 iOS `CodingKeys`／`rejectUnknownKeys`。Strict patch 否則整包拒收後走 REST；full Session reader 會忽略未知欄位。上游 [#2102](https://github.com/tiann/hapi/pull/2102)、[#2177](https://github.com/tiann/hapi/pull/2177) 均合併，運行中版本另查。
