---
title: HAPI inactive lease 不能代替 runner 程序退出確認
scope: tools/hapi
status: verified
updated: 2026-09-09
---

Hub `Session.active` 是心跳 lease；30 秒失聯即可變為 false，並不證明 detached runner child 已退出。新 lifecycle 操作不能只依賴 inactive 就 archive 或 delete，否則程序可能仍在跑，Hub 紀錄卻已消失。

`metadata.startedBy === 'runner'` 和 `metadata.startedFromRunner === true` 都表示 runner-backed session。重用 `SyncEngine.stopSession` 的 machine-scoped `stopRunnerSession` 確認：只有 `stopped` / `already_gone` 才能刪除；`still_alive`、RPC 離線、缺少 machine id 或不合法回應都應保留紀錄。

刪除保護放在共用 `SyncEngine.deleteSession`，讓 CLI DELETE route 與其他 engine callers 都受保護。呼叫停止前仍拒絕 active session；`SessionCache.deleteSession` 的再次 active 檢查處理等待 RPC 時重新連線的競態。已明確確認退出的內部 cleanup 流程需逐一檢查，不要僅憑名稱批次修改。

驗證：HAPI PR #1771 的 lifecycle suite 48 tests 通過；新增測試先在舊實作失敗，再驗證兩種 runner metadata、停止成功、仍存活、RPC 不可達、await 前不可刪除，以及 active-session guard。完整 gate：7,261 passed、5 skipped、0 failed。
