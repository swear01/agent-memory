---
title: HAPI remit 結果必須依 native terminal outcome 判定
scope: tools/hapi
status: verified
updated: 2026-09-09
---

`Session.thinking === false` 不是成功證據：heartbeat expiry、abort、native error 都可能清除 thinking。`wait-peer` 必須在該 remit 的訊息視窗找到明確 native success；部分 prose 加 idle 不能回傳 completed。`ready` 也不是成功事件，部分 launcher 在失敗後仍會發送。

`MessageQueue2` 可將同模式的多個 user message 合成一個 prompt，Hub 對同批 consumed local IDs 使用同一 `invokedAt`。結果擷取應略過相同 invokedAt 的其他 user row，以及尚未 invoked 的排隊 row；不同 invokedAt 的已消費 row 才是下一批邊界。

保留 runtime 現有終止原因，避免只靠 UI 的 thinking/ready：ACP 的 `AgentMessage.turn_complete.stopReason` 原先被共用 converter 丟掉；Claude SDK result 轉為 `system/turn_duration/resultSummary` 時須保留 subtype/is_error。Codex 須先通過 child/stale-turn 與自動重試過濾，再發送持久化終止訊息。Cursor 的 recovered stderr/inline error 也可能伴隨 end_turn，須等自動重試流程決定後才發布結果。

Pi `turn_end` 是 LLM/tool-loop 小回合，不能當整個 prompt 結束；等 `agent_settled`（或既有 legacy settlement）並 flush 文字後才發布最後 stopReason。AGY 須等 child close 與 send queue 排空；一旦原生 prose/tool/result 證明接收，先 emitMessagesConsumed，再輸出回答，否則 invokedAt 排序可能把回答排到 remit 前面。

驗證來源：PR #1771 的 wait-peer 回歸涵蓋 heartbeat expiry、idle、abort/error、同批 remits、Claude 空輸出成功；runtime 回歸涵蓋 Codex stale/retry、Cursor retry、Pi tool-loop settlement 與 AGY ack/output/terminal 順序。這些規則描述已實測的修正，不能用來假定尚未驗證的 runtime 結束事件格式。

中途 steering 是另一種同回合輸入：它的 `invokedAt` 不同於原 remit，不能當下一個普通 prompt。Hub schema v27 將 CLI `messages-consumed.steered === true` 存為訊息的 `steered` 欄位，REST、fork 與歷史合併也保留；不能從使用者請求的 `meta.deliveryMode` 推定已接受。Cancel probe 可能先寫 invokedAt，再收到正式 steer ACK，此時補上 steered，仍保留首次 invocation timestamp；後續普通 ACK 不可清除它。遷移及 socket→store→messages API／copy 回歸已通過。

`AcpSdkBackend.prompt` 的 `session/prompt` RPC 拒絕時，原本沒有 stopReason；launcher 捕捉錯誤並保持 active，會讓 wait-peer 等到 timeout。共用 backend 的 catch 將 stopReason 設為 error，重新拋出原錯誤，既有 finally 排空輸出後發布失敗 terminal。Cursor 已在外層延後 terminal 到重試決策完成，因此中間重試不會誤報失敗。新增 producer 與 steering 回歸在舊程式碼皆失敗，修正後通過。

Pi 文字輸出也是 cumulative snapshot：Hub row id 每筆不同，但 nested `data.id` 穩定且 `streamSnapshot: true`。`wait-peer` 與 `inspect-peer` 應按此 stream ID 更新同一文字區塊，保留第一次出現的位置與最新內容，不可把每個前綴都加入答案，也不可按文字內容去重。`wait-peer` 的 map 必須涵蓋整個 remit 視窗，不能逐頁重建；一般非 snapshot 訊息即使 data.id 相同仍需保留。兩個新增回歸在舊 collector 失敗、修正後通过，涵蓋跨頁、不同 stream ID、最新 row metadata 與普通文字。
