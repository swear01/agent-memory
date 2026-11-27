---
title: HAPI 在線卻停止工作時，先查 native terminal outcome
scope: tools/hapi
status: active
updated: 2026-11-28
---

`inspect-peer` 的 active、thinking、lifecycle 和底層 turn 結果是分開的。session 仍在線且摘要有文字，Codex 當輪仍可能已被中止。用該 turn 的原始 `event_msg/task_complete`、`error.codex_error_info`、程序與產物確認實際結果。

- 模型與 effort 讀同一 turn 的 `turn_context`。曾出現 worker 自述 Sol/high，實際兩輪都是 Luna/medium；全域設定和自述都不足以證明那一輪用的模型。
- `cyber_policy` 只證明 provider 中止該輪。保留現有事件和產物，釐清合法工作範圍，必要時走 provider 的誤判處理。不要自動重試或換 provider 繞過。
- Coordinator 明確交代 worker 直接執行，核對負責人、產物與驗證。被阻塞時記下恢復條件；worker 已完成且交接後，依 session 規則封存。記憶、Git 同步與 QMD 回讀各自驗證，見 [共享記憶流程](../../global/workflows/shared-memory.md)。
- 若背景工作收到 Jobs 的 HTTP 404，先核對 live Hub／CLI 的支援。已驗證 0.29.0.6 維護版曾移除 CLI/API Jobs 但保留 legacy schema；舊二進位和資料列仍可能存在。依現行支援決定操作，不用假 heartbeat 充當 attached job。

新 session 的核准模式見 [目前使用者偏好](session-permission-preference.md)。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/tools/hapi/terminal-errors-and-memory-closeout.md)
