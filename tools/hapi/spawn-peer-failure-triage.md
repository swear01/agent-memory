---
title: HAPI spawn HTTP 502 與 not found without verified exit 的不同根因
scope: tools/hapi
status: active
updated: 2026-11-28
---

- **`spawn_failed: HTTP 502`。** 先對齊 runner log 的時間與 session ID。已重現 `Worktree creation failed: Path is not a Git repository` 時，`--session-type worktree --dir` 必須指到 git repo。單看 502 不能定位 Hub、relay 或 runner。碰到探測 401/404，先核對 CLI 實際連的 Hub URL 與 API 版本。
- **Session 顯示 `active:false`，Runner 卻反覆印 `not found without verified exit`。** 這表示缺退出證據。0.30.x 的這個案例 `stop-peer` 回 500；0.35 已移除該 CLI。先查所有重複 ID 並核對 PID、start time、runner 記錄與實際存活證據。查大目錄用 `du` 或 `os.scandir`，避免列出數百萬條。
- 重啟 runner 本身不會補退出證據。歷史修復在證明特定舊程序已退出、並備份 state 後，補其 verified-exit tombstone，再用保留 child 的 supervisor 設定重載。程序身份或退出有疑問時保留原 state，見[退出查證](session-lifecycle-exit-confirmation.md)與[監督程序操作](supervisor-safe-operations.md)。
- 測試收尾只處理自己建立且已交接的 session。Archive 完要讀回 lifecycle；刪除另看授權。清工作樹前查 Session CWD、dirty 狀態及 branch 歸屬，留下未驗證或有工作中的樹。0.35 請用 [peer CLI 對照與 archive 工具](peer-cli-0-35-command-changes.md)。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/tools/hapi/spawn-peer-failure-triage.md)

- **Runner 在線，但 Codex child 立即 exit 1。** Hub 200、runner heartbeat／PID／控制埠正常，只證明 transport 路徑。沿 child 實際 launcher 收 stderr、cwd 和權限，分別報告 runner liveness 與該 session 可用性。根因不明時，重啟不是驗證。見 `git show 2505b5e:tools/hapi/runner-liveness-child-start.md`。
