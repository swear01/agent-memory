---
title: HAPI spawn 失敗：HTTP 502、ENOENT 與未證實退出
scope: tools/hapi
status: active
updated: 2026-11-28
---

- **`spawn_failed: HTTP 502`。** 先對齊 runner log 的時間與 session ID。已重現 `Worktree creation failed: Path is not a Git repository` 時，`--session-type worktree --dir` 必須指到 git repo。單看 502 不能定位 Hub、relay 或 runner。碰到探測 401/404，先核對 CLI 實際連的 Hub URL 與 API 版本。
- **`posix_spawn ENOENT`。** 分別查 launcher 可執行檔和該 session 的 cwd。歷史案例中 `git` 可執行但 session cwd 已不存在。以 exact runner/cwd 核對，其他路徑執行成功不能代替。根因為歷史回報，未證明後續恢復。[ENOENT 原案](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/spawn-enoent-working-directory.md)
- **0.30.7.2／.3 維護版回 `Existing HAPI session has no Codex thread binding`。** Carry #1771 曾把尚無原生 thread 的 remit stub 當成 cold resume；0.30.7.4 有經驗證的修正，0.35 已無這套 atomic spawn。當前遇 502 或 pre-exec cleanup 問題仍先核對版本、row identity、驗證過的退出與 child 是否實際啟動，不能照過期版本流程套用。[0.30.x 完整歷史](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/codex-spawn-remit-thread-binding.md)
- **Session 顯示 `active:false`，Runner 卻反覆印 `not found without verified exit`。** 這表示缺退出證據。0.30.x 的這個案例 `stop-peer` 回 500；0.35 已移除該 CLI。先查所有重複 ID 並核對 PID、start time、runner 記錄與實際存活證據。查大目錄用 `du` 或 `os.scandir`，避免列出數百萬條。
- 重啟 runner 本身不會補退出證據。歷史修復在證明特定舊程序已退出、並備份 state 後，補其 verified-exit tombstone，再用保留 child 的 supervisor 設定重載。程序身份或退出有疑問時保留原 state，見[退出查證](session-lifecycle-exit-confirmation.md)與[監督程序操作](supervisor-safe-operations.md)。
- 測試收尾只處理自己建立且已交接的 session。Archive 完要讀回 lifecycle；刪除另看授權。清工作樹前查 Session CWD、dirty 狀態及 branch 歸屬，留下未驗證或有工作中的樹。0.35 請用 [peer CLI 對照與 archive 工具](peer-cli-0-35-command-changes.md)。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/tools/hapi/spawn-peer-failure-triage.md)

- **Runner 在線，但 Codex child 立即 exit 1。** Hub 200、runner heartbeat／PID／控制埠正常，只證明 transport 路徑。沿 child 實際 launcher 收 stderr、cwd 和權限，分別報告 runner liveness 與該 session 可用性。根因不明時，重啟不是驗證。見 `git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/hapi/runner-liveness-child-start.md`。
