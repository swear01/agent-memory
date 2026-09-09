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
