---
title: Codex spawn-peer 將預留的新 session 誤當 cold resume
scope: tools/hapi
status: verified
updated: 2026-11-18
---

## 0.30.7.x 維護版的 Codex atomic spawn

0.30.7.2／.3 的 `spawn-peer --agent codex` 遇到 HTTP 502、runner 記 `Existing HAPI session has no Codex thread binding` 時，先核對版本與 transport。這個錯誤來自 carry #1771 的 `spawnSessionWithRemitOnce`，當時 stock upstream main 沒有該路徑。0.35 已移除這套 atomic spawn 與舊 wait/archive 命令，見 [peer 指令對照](peer-cli-0-35-command-changes.md)。

- Hub 為 remit 預先建 HAPI row，尚無 `codexSessionId`，runner 卻把它傳成 `--existing-session-id`；shared frontend 當成 cold resume。0.30.7.3 有 `reservedSessionId` 也不足以修此路徑，因為它仍傳 existing ID。
- 保留「接管新 row」和「續接有原生 thread 的舊 row」的分流。讓 shared runtime 的 initial create 沿既有 bootstrap 採用原 ID，僅符合 runner 啟動、pending remit、無 hostPid、未 archived 的新 row 放行。`/new`／fork 仍另建，不重用 remit identity。Hub 保留 server-owned metadata。
- 目錄不許可時，要保留機器端防線，且由 runner 的 `validateDirectory`／spawn 去重流程記錄 verified-exit tombstone。只有可證明未啟動的 child 才回 `childStarted:false`；只可清掉未被接管且 inactive 的 stub。`still_alive` 或 `unknown` 都不能刪；runner 重啟遺失 ID 時按 [退出查證](session-lifecycle-exit-confirmation.md) 走 guarded PID 探查。
- 驗證覆蓋真實 Hub→Runner→runtime 的 spawn、同 remit retry、wait、archive 與目錄拒絕，模型端是 mock。需要 opt-in runtime integration 的 MockSession 具備 EventEmitter。既有 terminal-owned engine attach timeout 在未修改基準也發生，不能算本修正結果。
- 0.30.7.4 carry #1771 `5a0465f6` 後，Mac 與 mazu 的 native spawn、回覆和 archive 讀回通過；Windows swop 當時未驗。切到其他 release 時查實際 code/manifest，不能沿用此舊驗證。

狀態、CI 和修正原始記錄見 `git show 2505b5e:tools/hapi/codex-spawn-remit-thread-binding.md`。
