---
title: Claude Code stream-json 現在支援 mid-turn steer，HAPI 仍把 Claude 排除在 Steer 之外
scope: projects/hapi-claude-midturn-steer
project: hapi
status: active
confidence: high
evidence: 2026-10-21 用 claude 2.1.302 (--input-format stream-json, haiku) 實測；第一個 tool_use 後注入 user message，turn 在下一個 tool boundary 轉向、跳過後續步驟、只產生一個 result。
created: 2026-10-21
updated: 2026-10-28
tags:
  - hapi
  - claude
  - steer
  - stream-json
---

- tiann/hapi #888 的結論「stream-json mid-turn stdin 會被忽略」是在 claude 2.1.175 驗證的，已過時。
- 2.1.302：turn 進行中寫進 stdin 的 user message，只要後面還有 tool boundary，就會在那裡被吃進「同一個 turn」（不會多一個 `result`，stream 也不會 echo 這則 user message）。如果送進去時 Claude 已經在寫最後的回覆、後面沒有工具呼叫了，它會在原本的 `result` 之後另開一個 follow-up turn（先 `system init`，再多一個 `result`）處理。訊息不會丟，但 wrapper 在第一個 `result` 送出的 ready 會太早。
- HAPI 沒有 Claude Steer 的原因：`shared/src/modes.ts` 的 `STEERING_SUPPORTED_FLAVORS` = codex/cursor/pi；`cli/src/claude/claudeRemote.ts` 只在 `result` 之後才 `scheduleNextMessage()`，所以 web 送的訊息會一直留在 MessageQueue2，等到 turn 結束。
- 要實作的話：把 claude 加進 gate，並在 claude 的 CLI 處理 `SteerQueuedMessage` RPC，turn 進行中直接 push 到 `messages` iterable（參考 codex/cursor 的 handler）。
- 已有上游 PR：tiann/hapi #2008（@Shalistera，2026-10-12 開，CI 綠燈，等 maintainer）。注意：bot review 只有 Requirement/Approach 是 Pass，latest head `4c22ab47` 的 Code review 仍有 Major×2、Minor×1（啟動期間 steer 沒追蹤、tool-abort 提早 return 時沒先停掉舊 process 就還原 steer、in-flight cancel 誤刪 echo marker）。不要只看 "Pass" 字樣。內容涵蓋上面的實作，另外用 prompt `uuid` 配合 `command_lifecycle` 處理「steer 變成 follow-up turn」時 ready 太早送出的問題。不要再另開重複的 PR。
- 脈絡：swear01 自己開的 #888（codex+claude）在 2026-08-19 被 #1606（Codex）合併時自動關閉，Claude 的部分沒做，也沒有另開 Claude 專用 issue。另一個競爭 PR #1351（@yin1245，預設自動 steer、取代排隊）已經 CONFLICTING，從 2026-10-03 起沒更新；#2008 沿用逐則 Steer 按鈕，符合 #888 和 #1466「預設排隊」的取向，但 PR 說明沒有連到 #888。
- 結論（2026-10-22）：不另開上游 PR。使用者要求維護版要有，所以 v0.30.7.6 候選以 overlay 方式帶 #2008 + 修三個 finding 的 patch（swear01/hapi#30）。claude 2.1.302 實測：帶 uuid 的 prompt 會在 `result` 之前送出 `command_lifecycle` queued/started。本地 worktree `<worktree-root>/hapi/claude-midturn-steer-20261022` 是未 commit 的試作，可以刪。
- 2026-10-23 狀態：fork release PR swear01/hapi#30（v0.30.7.6，含 #2008 overlay）仍 OPEN 未發佈；上游 #2008 仍 OPEN，head `4c22ab47`，從 2026-10-13 起沒有更新。
- 2026-10-28 Steer UX 缺口（v0.30.7.7）：Claude steer RPC 寫進 stdin 就回 `steered:true`，但要等 `command_lifecycle started`（下一個 tool boundary，或 `result` 之後）才 consumed；這段期間 web 仍是普通 Queued 列，可再按 Steer（"Message not in queue"）、Cancel/Edit（hub busy → "Delivery outcome unknown"）。原因：Claude handler 沒像 Codex/Cursor 呼叫 `setSteerDeliveryState('dispatching')`。
  - 共用 UI 不用另修：上游 #1981（OpenCode，junmo-kim）2026-10-28 head `3b70fdd` 加了不分 flavor 的 `deliveryPending`（"Waiting for delivery confirmation"、按鈕 disabled、reload 保留）。下一版 fork 要 carry #1981 最新 head，舊 carry 沒有這段。
  - Claude 修正：本地 branch `fork/claude-steer-dispatching` commit `6e756a952`（base release/v0.30.7.7），下一版當 regenerate overlay patch 放在 0055（#2008 fix）之後。pre-push hook 擋非 upstream/main 為基底的 push，所以沒推；fork 修正本來就走 manifest patch，不靠遠端 branch。重點：restore 要先同步放回本地 queue、再 fire-and-forget 寫 `queued`（hub 有 `invoked_at IS NULL` guard），否則 turn 結束時可能讓後面的訊息先跑。已在 #2008 留言附 diff。
  - Cursor ACP（#1609 swear01）：abort 後 soft steer 回 `stopReason: cancelled` 卻被當成已送達。修正 PR tiann/hapi#2229（head 9ec0c73f HAPI Bot pass、CI 全綠，等上游合併）：cancelled → indeterminate（不自動重送，避免重複/Stop 後自己跑）。「row 永遠卡 dispatching」經查在 upstream/main 不成立。
  - #1981 留言：`nativeSteerReceiptCorrelationLost` 永不重設，一次 Stop 之後該 launcher 的 steer 都退回整個 turn 完成才確認。
