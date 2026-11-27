---
title: HAPI remit 結果必須依 native terminal outcome 判定
scope: tools/hapi
status: verified
updated: 2026-11-28
---

先區分 HAPI session 暫時閒置、某次 remit 已終止、任務交付已完成。新版用 `hapi inspect-peer` 或該版本支援的查詢。0.30.x 維護版的 `wait-peer` 已在 0.35 移除。[peer CLI 版本表](peer-cli-0-35-command-changes.md)

- **`thinking=false` 或 `ready`。** 核對所要訊息的 native terminal outcome。Heartbeat 過期、abort、native error 會讓 UI 閒置，有些 launcher 失敗後也會送 ready。PR 等待與外部背景工作還要讀回 PR、測試或產物。
- **排隊與訊息邊界。** 同批 `MessageQueue2` 的 consumed local IDs 共用 `invokedAt`，超過這批的已消費普通訊息才是後一個邊界。未 invoked 的 queue row 和同輪的 `steered` 訊息不截斷原 remit。Hub schema v27 用 `messages-consumed.steered` 保存是否接受，晚到的 steer ACK 補欄位並保留第一次 `invokedAt`；普通 ACK 保留既有旗標。
- **各 runtime 的終止事件。** 保留 ACP `stopReason`、Claude result `subtype/is_error`。ACP prompt 拒絕經 finally 排空輸出後發 error；Cursor 和 Codex 先排除重試或 stale turn。Pi 等整個 agent settlement，`turn_end` 可能只是內部的工具迴圈。AGY 等 child close 與 send queue 排空；原生輸出證明已接收時，consumed ACK 要早於該回答。
- **Pi snapshot 與用量警告。** `streamSnapshot: true` 按 nested `data.id` 在整個分頁視窗保留最新文字與第一次位置。普通訊息照常保留。用量上限會留下 `lifecycleState=running`、`thinking=false` 和用量警告。額度重置後查原工作樹與背景副作用，續跑同一個 session 並讀回新的 remit；有多個 worker 時逐一核對。
- **只擷取到 user 文字或流程中斷。** 分別確認 Hub 已保存的訊息、runner 本機狀態與真正產物。技能 canonical 比對失敗是獨立的啟動錯誤。若原工作會寫外部文件，先讀回已有內容再考慮重試。

[原始回歸、Pi 交接與等待案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/session-remit-native-outcomes.md)
