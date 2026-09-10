---
title: HAPI updates and restarts must preserve child sessions and fail closed
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated rollout rehearsals, supervisor policy checks, exact-process recovery, cleanup ordering, platform verification, and a live Mac session-recovery check on 2026-08-20.
created: 2026-08-18
updated: 2026-09-10
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

# Restart contract

Restart only the supervisor's main runner:

- Linux systemd: `KillMode=process`
- macOS launchd: `AbandonProcessGroup=true`
- PM2: `treekill=false`

Do not manually kill runner-created children. Do not use broad process matchers.

# Handoff recovery

If a version handoff leaves a duplicate main runner, read the state PID, verify its command line is exactly the expected main `hapi runner start-sync` process, and stop only that PID.

After recovery, verify supervisor `MainPID`, state PID, restart stability, API identity, and the pre-update child-session baseline.

# Update safety

Use `set -o pipefail`; nested update or restart failures must propagate non-zero. Write installer downloads to a user cache, not a shared or privileged temporary directory.

On shared-filesystem hosts, publish a shared binary once, then verify each runner before restarting the hub.

# Cleanup safety

Operate only on the exact registered machine identity and back up the hub database first. For connected sessions:

1. Archive or terminate through the Hub.
2. Use the runner `stop-session` endpoint.
3. Send TERM to an exact PID only after confirming the process is disconnected.

Directly killing a connected orphan can trigger CLI cleanup and create misleading hub sessions.

# Disconnected-session recovery on swairM5

For the Mac display name `swairM5`, treat an inactive session with `metadata.lifecycleState=running` as a disconnected-session candidate; do not bulk-revive archived sessions. Exclude the current session, then process each candidate in order:

1. `POST /api/sessions/:id/resume` with `{}`.
2. Use the returned `sessionId` (it may differ after merge) for `POST /api/sessions/:id/messages` with `{"text":"繼續","localId":"<fresh-uuid>"}`.
3. Verify `GET /api/sessions/:id` reports `active=true` and the message page contains that `localId`.
4. Check the exact process through `hapi runner list` and `hostPid`; do not infer active work from the database row alone.

`active=true` confirms a live session connection, not an actively generating turn; `thinking=false` means the agent is currently idle or waiting. A resumed session can run, emit agent messages, and later archive, so report those states separately. Keep the recovery namespace-scoped and retain no raw transcript or token in memory.

# Session lifecycle and group actions

`active=false` is not the same as archived. A row with
`metadata.lifecycleState=running` and `active=false` is a split-brain cleanup
candidate, not permission to revive it. Archive operations should be
idempotent for already-archived rows, and every session must be formally
archived before a group deletion. Apply unsupported-resume guards before an
active-session handoff.

# Platform checks

Verify the loaded supervisor configuration, signing identity where applicable, protected-folder probes, and the platform's restart command after every rollout.

# Verified maintained-release baseline (2026-09-06)

HAPI `v0.29.0.6` 已發布並部署到 standalone Hub 與全部 8 台 Runner；main/tag 為 `cc45252848ba0033d1927e24c1121f821eb5b68d`。Release run `34038209117`、main/tag CI、9 assets 的 8 payload SHA-256、兩種 macOS `xyz.hapi.cli` strict signing 均通過。這是部署完成紀錄，未來操作仍應重新讀取 live 狀態。

- `.6` 依 operator 要求暫時排除 #1424 session-attached Jobs 的 CLI/API/meters/Jobs 置頂；保留 schema V29、legacy job rows、正常 active-session pinning，以及原先混在該補丁中的獨立 CLI/Cursor 修復。
- 5 台 Linux 與 Oracle 都曾出現新版 Runner 在線，但 systemd activating/restart-loop 或 PM2 errored 的 handoff。僅驗證 binary version 或 Hub heartbeat 不夠。先確認 state PID 的 argv 是主 `runner start-sync` 且沒有 `--started-by`，再停止 supervisor、TERM 該主 PID、啟動 supervisor；Linux 要 MainPID 等於 state PID 且重試穩定，Oracle 要 PM2 online、treekill=false、PM2 wrapper 實際為 Runner 父程序並 pm2 save。
- session 保留用 PID 加 start time 比較。此次第一階段 12/12 保留；最終 2 個 Zeus session 明確有 `archiveReason=User terminated`，剩餘 10/10 PID/start time 不變。不可把稍後使用者終止算成部署殺掉，也不可無證據排除遺失程序。
- 最後回查 standalone Hub/channel off/policy alert、Hub/Tunnel HTTP 200、served Web `.6` 與 no-store、DB V29 quick_check ok，以及 Mac protected-folder Runner RPC。

# Exact-source Windows artifact cache

Windows 定向升級的 source generation 必須對應正式 release source；artifact SHA-256、offer、實際新 Runner PID/version/generation 都要一致，`started` 回覆不算完成。

Mazu 暫時 source Hub 使用快速本機磁碟與完整 fingerprint inputs，並包含 `hub/dist`、`web/dist`、pinned tunwg platform binaries。WorkingDirectory 設 source tree 的 `hub`，不是 `hub/dist`。ensureCliArtifact 在 cache lookup 前仍檢查／下載 tunwg；唯讀快取缺少該 binary 會先因 EACCES 失敗。完成指定 Windows 升級後，移除本次專用 drop-in、恢復 standalone Hub，保留 DB/env 與其他 drop-ins。

# Reboot PID reuse and stale lock (2026-09-10)

Athena reboot 後，`runner.state.json` 與 `runner.state.json.lock` 的舊 PID
已被無關的 `deepseek-gateway` 重用，舊 control port 也沒有 listener。HAPI
0.29.0.6 仍印出 `Runner already running with matching version`，systemd
反覆 auto-restart，而 Hub 找不到該機器。只看 PID 存在會誤判 runner 存活。

恢復時先確認目前 boot、PID 的 `/proc/<pid>/exe`/argv、控制埠及是否有真正
runner；不能對 state 記錄的 PID 直接 kill。停止 runner supervisor 後，保留
並移走已驗證過期的 state **及文字 PID lock**，再啟動 supervisor。這次只移走
state 不夠，lock 仍會擋啟動。無關程序全程保留；最後實際 runner PID、state PID
與 systemd MainPID 相同、NRestarts=0、Hub 上線且原 coding sessions 能 resume。

主機重啟前未完成的 capture 仍是中斷，恢復 HAPI 不會恢復 verifier 程序。
保留原 artifacts 與 RUNNING receipt 的原始 bytes，另外記錄 interruption，再給
新 attempt/輸出目錄與 prospective admission。主機為何重啟仍未知；此鎖定問題
只解釋重啟後 runner 無法恢復。永久程式修補另追蹤 CPAchecker issue #252。
證據：`<experiments-root>/reports/next-parallel-batch-20260910/athena-runner-recovery.json`。
