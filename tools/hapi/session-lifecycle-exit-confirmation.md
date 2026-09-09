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

Hub 已保留 child session id 的 spawn 請求，即使在真正建立 child 前被拒絕，也必須留下可供 cleanup 使用的退出證據。`spawnSessionOnce` 的 `failBeforeChild` 會記錄 verified-exit tombstone；machine RPC 若提前因 workspace roots 拒絕，便跳過此 bookkeeping，造成沒有 PID 也沒有 tombstone，`stopSession` 只能回 still_alive，預留 row 無法封存。保留傳給 runner 的 `validateDirectory` callback，讓同一個 pre-child rejection 路徑完成驗證和退出記錄；不要靠刪除資料庫紀錄來繞過退出確認。

驗證：新增真實 Hub → machine RPC → runner integration regression，舊實作回 `outside_workspace_roots` 且 `cleanedUp:false`；修正後 `cleanedUp:true`、預留 session 已 archived、stop 回 already_gone、沒有 child 或被拒絕的目錄。完整 serial runner integration：15 passed、1 skipped，測試所屬程序清理 audit 通過。
