---
title: HAPI updates and restarts must preserve child sessions and fail closed
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated rollout rehearsals, supervisor policy checks, exact-process recovery, cleanup ordering, platform verification, and a live Mac session-recovery check on 2026-08-20.
created: 2026-08-18
updated: 2026-10-26
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

`printf script | ssh host bash -s` 結尾若呼叫 `hapi --version`，compiled binary 會讀剩餘 stdin 並把 SSH 掛住；遠端其實可能已裝完。細節見 `tools/hapi/remote-install-stdin-hang.md`。Hub 健康檢查的 `curl` 必須設 `--max-time`，否則半開的 `:3006` 會讓 SSH 連十數小時。腳本與 gist 預設仍是 `0.29.0.6`，尚未跟著 `0.30.4.1` 改。

共享 NFS binary 換檔後，仍在跑的 `hapi hub` 會繼續用 `.nfs*` inode。HTTP 200 不夠；要看 `/proc/<pid>/exe` 是否已指向 `<remote-home>/.local/bin/hapi`，以及 served JS 是否含新版號。Hub DB 約 4.5–4.8 GiB，不要在長 SSH 上 `cp -a`；用 SQLite online backup，見 `tools/hapi/hub-sqlite-lock-crash.md`。

zeus 的 canonical machine id 是 `aa6fb76d-ca35-46c1-aaea-42a5b1c04d06`（不是 `aa6fb76b-...`）。

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

# Verified maintained-release baseline (2026-09-15)

HAPI `v0.30.4.1` 已發布；Unix 7 機與 standalone Hub 於 2026-09-15 驗證為該版。維護 SHA / tag / `origin/main` 為 `76d578dcaa6c8dd7cc0d3a6ce56eb2513da397c7`，官方 pin `v0.30.4` = `8cefb0f04c413d4f47cd364bcf556e03af6b4073`。Release run `34822868886`、tag Test `34822868907`、8 payload SHA-256、macOS `xyz.hapi.cli` 通過。Windows `swop` 此輪未部署。這是部署完成紀錄，未來操作仍應重新讀取 live 狀態。

NFS 四機加 zeus / oracle / Mac 都在 binary mtime handoff 後進入 `Runner already running with matching version` 迴圈。只看 binary `--version` 或 Hub `running` 不夠。先確認 state PID argv 是主 `runner start-sync` 且沒有 `--started-by runner`，再 stop supervisor、只 TERM 該 PID、start；Linux 要 `MainPID` 等於新 state PID 且 `NRestarts=0`，Mac launchd 要 `running` 且 job PID 等於 state PID，oracle 要 PM2 wrapper 為 Runner 父程序、`treekill=false`、`pm2 save`。

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

程式修正邊界：HAPI [PR #19](https://github.com/swear01/hapi/pull/19) 已合併於
`806a39190a83242dc670168f32bd11840a266947`，但此時尚未 release／部署。
`acquireRunnerLock` 改用既有 `getHapiRunnerProcessIdentity`：只回收 `dead`／
`foreign` lock owner，`runner`／`unknown` 保留占用，重試仍有界；12 個 focused
測試、CLI typecheck、兩組 CI 與零 finding review 通過。既有 state same-version
檢查本來就使用該 identity helper，這次 lock delta 沒有證明原部署為何誤走
same-version 分支；installed binary／probe unknown 的歸因仍未解。合併不等於
各機器已安裝修正版，也不代表主機重啟原因已解決。

# 權限提示重現不能以 runner 重啟結案

歷史對話中，Agent 根據 TCC 已允許、runner 早於權限更新啟動，推測舊程序保留權限狀態，報告已重啟且原 session 仍存活，並預期新 session 不再彈窗。使用者隨後明確回報「還是又跳出來了」。因此舊程序原因仍未確證，重啟也未證明解除症狀。

保留既有 session，針對再次觸發的操作核對實際程序、權限對象與相關 log；用症狀是否重現驗證修補，不以新 PID 或 runner running 代替。這個案例不支持把「先重啟」當成通用處方，也不支持未查原因就重設 TCC。

來源：Issue25 backlog consolidation v1，controller-publication-v1 的 permission-recurrence 對照紀錄；正文與兩則相鄰訊息已核對。此處只記錄歷史失敗與驗證邊界，沒有宣稱當前仍有問題或已修復。

# v0.30.7.6 rollout lessons (2026-10-24)

- mazu 沒有 `sqlite3` CLI；Hub DB 備份用 Python `sqlite3.Connection.backup`（同一個 online backup API），再對備份跑 `quick_check` 與 `user_version`。`.6` 把 Hub schema 27 升到 29，舊版 Hub 讀不了。
- `hapi spawn-peer --agent codex --permission-mode auto` 會回 `bad_args: permission mode auto is not supported by codex`。Codex 可用 `default`/`read-only`/`safe-yolo`/`autoReview`/`yolo`；「auto」對應 `autoReview`。
- 部署後 mazu 第一次 Codex spawn 回 HTTP 502；runner log 是 `spawnSync <remote-home>/.local/bin/codex ETIMEDOUT`。之後 `codex --version` 冷啟動 1.1 s、熱啟動 0.12 s，重試一次就成功。失敗的 child session 已自動 archive。
- Windows swop：`hapi.exe` 執行中可以 rename，所以先把舊檔改名，再複製已驗 SHA-256 的新 exe。舊 Runner 偵測到 mtime 變化後會自行 handoff，但新 Runner 的父程序是已退出的舊 PID，等於脫離 Scheduled Task 鏈（task 每分鐘回報 `Runner already running with matching version`）。確認 state PID 是主 `runner start-sync`、且沒有 `--started-by runner` 程序後，只 `Stop-Process` 該 PID 並 `Start-ScheduledTask`，就恢復 wscript→powershell→hapi 鏈。`LastTaskResult 0x800710E0` 是 `IgnoreNew` 的正常結果。
- Retention：舊規則是 Athena 無法檢查時保留 NFS 共用的舊 runtime／`.nfs*`。2026-10-24 使用者指示「Athena 不要管了」後改為只查 mazu／cthulhu／valkyrie 的 exe／cwd／fd／maps；兩輪檢查加上以已知持有者（mazu 9586 的 `.nfs…e1524ea`）做正向對照後，刪除 0.30.7.4 runtime 與 6 個舊 `.nfs*`，約 1.37 GB。盤點時不要執行舊 `.nfs*` binary 的 `--version` 來判斷版本，那會違反只讀；版本只能靠 mtime 推斷。

# 升級後的舊 session 遷移（2026-10-24 起為常設規則）

- 使用者要求每次升版都把仍跑舊 binary 的 session 移到新版，並直接刪掉舊版。做法：從 `--started-by runner` 程序的 cmdline 取得完整 `--hapi-session-id`。等 `hapi inspect-peer` 間隔一分鐘的兩次檢查都是 `active=true`、`thinking=false`，再執行 `hapi stop-peer`，這是 process-only stop，不 archive。下次從 Web 送訊息時，Hub 會以新版 binary 和 `claude --resume` 接回同一段對話。最後才重做 holder 檢查並刪除舊 runtime。
- Binary 用 `mv` 原子替換後，舊 session 的 `/proc/<pid>/exe` 會顯示 `(deleted)`（NFS 上則是 `.nfs*`），可以用這點找出所有舊版 session。這些 session 仍可能隨時使用舊版的 `runtime/<old>/tools`，所以遷移前不能刪那個 runtime。
- Agent 在未經使用者授權時對使用者 session 執行 `stop-peer`，會被 Claude Code auto-mode classifier 以 Interfere With Workloads 拒絕。使用者明確授權後，等 session 閒置才停的背景腳本就被允許。
- 2026-10-24 在 oracle 發現 11 個 2026-08-15 漏掉的 vitest 子程序（`bun …/cli/src/index.ts claude --started-by runner`），PPID 都是 1。它們的 `HAPI_HOME=/tmp/hapi-test-*` 已不存在，連的是已經沒有 listener 的 `127.0.0.1:<port>`，env 帶 `VITEST=true`，合計約 850 MB RSS。逐一核對 env、PPID 並確認沒有 TCP 連線後，以 exact PID 送 TERM 清除。

# 舊 binary 的 Mac session 一重啟 Claude 就 exit 126（2026-10-26）

- 症狀：Web 顯示 `Process exited unexpectedly: Claude Code process exited with code 126`，連續三次後出現 `Dropping the queued message`，訊息沒有送到 agent。hapi CLI 的 log 只有 `[remote]: launch error {}`；child stderr 只有在設了 `DEBUG` 時才會印出來，所以 log 抓不到 errno。
- 情境：swairM5 上執行 v0.30.7.7 部署的那個 session，在 03:45 用新版覆蓋了 `~/.local/bin/hapi`，但它自己還是跑舊 binary。它的 Claude child 從 03:40 一直活著，所以中間正常。14:37 切換 model 後必須重啟 child（log 出現 `mode has changed`），之後每次 spawn 都在約 40 ms 內 exit 126。同一個 process 稍早也回報過 macOS TCC 拒絕它讀取 `~/Documents` 裡的檔案。
- 原因調查（以 tccd log 為準）：
  - hapi 讀 Documents 靠的是 Full Disk Access（`kTCCServiceSystemPolicyAllFiles`，`authValue=2 authReason=4`）。TCC 用路徑認 client（`AUTHREQ_SUBJECT: subject=~/.local/bin/hapi`，`staticCode … type: 1`），designated requirement 是 `identifier "xyz.hapi.cli" and certificate root = H"4abaabe4…"`，不綁 cdhash。每版都用同一張 `HAPI Local Release Signing` 簽，所以**授權會延續到新版，新 process 都被允許**。換版本身不需要重新授權。
  - 舊 process 的不同之處：它的執行檔 inode 已經被 `mv` 換掉。tccd 處理它的請求時會記 `Failed to fetch responsible file descriptor: [2: No such file or directory]`，接著改成驗證「目前在那個路徑上的新檔」，而不是實際在跑的程式碼。新 process 不會出現這行，所以可以當作舊 inode process 的辨識記號。
  - 時間線：03:45 換掉 binary 以後，舊 process 到 04:02 都還讀得到 Documents（推測是 kernel 快取了舊的允許結果）。閒置 10.5 小時後，14:34 讀 Documents 得到 `Operation not permitted`，而且當下 sandboxd 和 tccd 都**沒有收到任何請求**。14:37:54 spawn 時 tccd 對舊 process 回的是允許，但 spawn 仍然 exit 126。所以拒絕發生在 kernel 這一層，不是 tccd 判 deny。推論是 kernel 無法把 tccd 的結果對應回舊程式碼的身分；這點沒有直接證實，因為重現需要替換正在使用的 binary，會連帶影響所有 live session。
  - 排除項目：不是 iCloud dataless（`ls -lO` 沒有 dataless 旗標，`FXICloudDriveDesktop=0`）；不是 Claude Code 的 sandbox（沒有 `sandbox` 設定）；不是 fd 耗盡；不是 env 或參數造成（用舊 process 啟動時的 env 和相同參數手動跑都正常）。
  - 調查陷阱：zsh 的 `log` 是內建指令，要用 `/usr/bin/log show`。`kTCCServiceDeveloperTool` 和 `ListenEvent` 的 deny 是 syspolicyd 等的 preflight 雜訊，跟這次無關。
- 結論：在 Mac 上，舊 binary 的 session 在換版後一段時間內還能用，但遲早會失去對 `~/Documents` 的存取，包括 stow 進去的 `~/.local/bin/claude`、`~/.ssh/config` 和 skills。之後只要需要重啟 Claude child，或讀到這些檔案就會失敗。Linux 沒有這個問題。切換 model 或 mode、遇到 crash 需要 respawn 時，就可能一直卡在 126，這時換回原 model 也沒用。
- 修法（已驗證）：等舊 process 停掉（或在授權下 `hapi stop-peer <id>`），然後執行 `hapi ping-peer <id> --message-file -`。inactive session 會先自動 resume（JSON 回 `"resumed":true`），由 runner 用新 binary 加 `claude --resume` 接回原對話，再把訊息送進去；被丟掉的訊息要手動重送。驗證方式：新 log 沒有 `launch error`、有 `stream message #`，`inspect-peer` 看得到 agent 回覆。
