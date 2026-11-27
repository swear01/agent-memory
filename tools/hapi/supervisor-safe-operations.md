---
title: HAPI Runner 交接、重啟與舊 session 的程序查證
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated rollout rehearsals, supervisor policy checks, exact-process recovery, cleanup ordering, platform verification, and a live Mac session-recovery check on 2026-08-20.
created: 2026-08-18
updated: 2026-11-28
tags:
  - hapi
  - supervisors
  - deployments
  - sessions
  - cleanup
source_refs:
  - public-release:supervisor-safe-operations
  - hapi-live:swairM5-session-recovery-2026-08-20
  - hapi:hub/src/web/routes/sessions.ts
  - hapi:hub/src/store/sessions.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

## 重啟與升級

- **Supervisor 重啟可能誤停 child。** 只重啟主 runner。確認已載入 Linux systemd `KillMode=process`、Mac launchd `AbandonProcessGroup=true`、PM2 `treekill=false`。先收現有 session PID 與 start time，重啟後逐一比對。Windows 另核 Scheduled Task 的 ownership。
- **`Runner already running with matching version` 進入 restart loop。** 用 supervisor PID、state PID、實際 exe／argv、控制埠區分 duplicate runner 和 reboot 後的 PID reuse。只在確定 split ownership 時停主 `runner start-sync`（不含 `--started-by runner`）再交回 supervisor。不相干 PID 即使出現在 state 也要保留；停 supervisor 後移開確認過期的 state 和文字 PID lock，再驗主 PID、`NRestarts`、Hub 與 child。修好 runner 不會復活中斷的 verifier。
- **HTTP 200 或 `--version` 正常，live Hub 還是舊版。** NFS 共享 binary 只發布一次。檢查 `/proc/<pid>/exe`、Web asset、schema 和 SQLite `quick_check`。DB 用 SQLite online backup；沒有 CLI 時用 Python `sqlite3.Connection.backup`，再查 `user_version`。升級腳本用 `set -o pipefail` 並限制健康探測時間。遠端 `hapi --version` 要避免讀部署腳本的 stdin。[stdin 問題](remote-install-stdin-hang.md)
- **Windows offer 說 `started`，升級仍未完成。** 核對正式 source generation、SHA-256、live PID／version 與父程序。mtime handoff 可能讓新版脫離 Scheduled Task；核對 exact state PID 才交回原 Task。重建 artifact cache 要保留完整 fingerprint、`hub/dist`、`web/dist` 和 pinned tunwg；ensureCliArtifact 在 cache lookup 前仍會查 tunwg。用臨時 source Hub 時，完成後只移除本次 drop-in 並保留 DB/env。

## Session 恢復與清理

- **Hub 顯示 inactive。** `active=false` 可能是斷線而非 archived。重開後用 `activeAt`、`metadata.lifecycleState` 的 running／idle、實際機器與最近訊息查受影響者。等機器重連，避開自己的 session、已封存或空白聊天。名稱列表可能截斷，核對 API 取得的完整範圍。僅按授權 resume。
- **Resume 回 success，但工作還沒繼續。** 用 resume 回傳的實際 session ID，可能因合併而改變。需要繼續工作時另送附唯一 `localId` 的訊息。驗證 Hub 連線、message 回讀、`hapi runner list` 的 hostPid 和實際輸出。重開 NIS master 故障時看 [Ubuntu／NIS 恢復](../../machines/hapi-fleet/ubuntu-upgrade-recovery-runbook.md)；`urllib` 使用預設 UA 曾被公開 edge 拒絕，先核實可靠的 Hub 路徑。
- **清理被斷開的程序。** 核對 machine 和 session，先備份 Hub DB。群組刪除前逐條讀回已 archive。單一程序終止先走 Hub 和 runner lifecycle；只有已證明斷線並核對 exact PID 時才考慮最後的 TERM。見 [退出查證](session-lifecycle-exit-confirmation.md)。
- **新版本正確，仍有舊 session 使用 runtime。** 原子替換後舊 Linux 程序的 exe 可能為 `(deleted)` 或 `.nfs*`。用 exe／cwd／fd／maps 查實際持有者，覆蓋不足時保留。0.30.x 曾可在授權下對已驗證 idle 的 session 用 process-only `stop-peer`；0.35 已移除，等原程序自己 inactive，再以新版 resume。舊 runtime 未確認無持有者前不可刪，盤點時也別執行舊 inode。

## Mac 權限與 Claude exit 126

- **本機 Mac Catalyst 測試的簽章。** 執行 HAPI 測試 App 前，確認本機可使用指定的固定簽署身分，分開 build-for-testing 與啟動，對實際產物執行 `codesign --verify --strict` 並核對簽署者及 designated requirement。正式 CLI 使用 `HAPI Local Release Signing`；GitHub 能簽 CLI 並不代表本機有可用私鑰或 Catalyst 所需權限。無法以指定身分簽署時回報原因。使用者未授權以 `CODE_SIGNING_ALLOWED=NO` 或 ad-hoc 簽章啟動本機 HAPI 測試 App。曾有兩份 `run.hapi.app` 測試產物因各自以 cdhash 識別，先後觸發「文件」權限提示；用 tccd 的 `AUTHREQ_PROMPTING` 核對 service、程式路徑與請求 ID，再判斷是否涉及正式 runner。
- Restart 成功後若仍跳 TCC 提示，直接重驗觸發操作與負責程序。重啟、簽署或讀回授權各自只證明當步驟。
- 舊 inode 的 session 在更新後曾報 `Process exited unexpectedly: Claude Code process exited with code 126`、`Dropping the queued message`。用 `/usr/bin/log show` 看 tccd 的 `Failed to fetch responsible file descriptor`，核對 launcher 是否經 symlink 指到 Documents。已驗證的預防是把 launcher 複製為實體檔；其餘受保護路徑仍要重驗。等舊 session inactive 後，用新版恢復並重新投遞被丟掉的訊息。0.35 的 `ping-peer` 需 `--wake` 才會恢復 inactive session；看到新版程序及 agent 輸出才算成功。

[整理前完整紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/tools/hapi/supervisor-safe-operations.md)
