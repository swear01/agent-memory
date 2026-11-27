---
title: HAPI maintained fork release, upstream review, and rollout pitfalls
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Audited patch replays, exact-head reviews, release gates, and live rollout checks.
created: 2026-08-18
updated: 2026-11-28
tags: [hapi, fork, upstream, release, permissions, rehearsal]
---

## 上游 PR、維護版和部署的授權邊界

- 更新既有 `tiann/hapi` PR 的授權限於該 PR 的 head 分支。搜尋、重現、開 issue、實作和發 PR 各有範圍。合併、auto-merge、關閉 PR、改 fork main、發版和部署均須依各自的明確授權。綠燈和 `MERGEABLE` 只代表可交給維護者。
- 先查 upstream main、既有 PR、實際維護版 tree。功能可能已被後繼 PR 或 squash 納入，即使原 PR 仍顯示 OPEN。維護版的 overlay bug 優先修造成問題的 PR 或 fork overlay，避免對不存在該 bug 的 upstream main 開 issue。向別人的 feature branch 提交補修不會更新原 upstream PR；從該 feature 的實際 head 建 publication branch，再測它的 exact tree，可避免夾帶驗證用的 upstream merge。
- 更新自己的 upstream PR 時，先比對 PR head 和遠端 branch SHA。用新的 upstream worktree merge 既有 head，可保留 ancestor 並 fast-forward push。既有使用者 worktree 不 reset。對 `base.ref=main` 仍 checkout 到舊 trusted workflow 的情況，檢查 `base.sha`。對同一 PR PATCH `{"base":"main"}` 曾刷新 base SHA 且不改 head；需要新的 review 時，用提交具體維護說明觸發 `edited` 或由有權限的 maintainer rerun。

## Carry、manifest 和 rehearsal

- 維護版從選定的官方 release base，加 audited queue 與維護版號 patch 重建。官方 release SHA 可以是 upstream main 的 ancestor。依當版 upstream 的 packageManager 對齊發版 workflow、測試和執行環境。若 script 內再呼叫 `bun`，調整 job 的 PATH 並核對實際 compiler；login shell 可能還原 PATH。官方 CLI `optionalDependencies` 的版本屬官方 base。
- Rehearsal 和正式推 main 前，都重新分頁抓 open PR、live head、review、維護者訊號和功能範圍，核對 audit/manifest。上游新增、關閉、rebase 或合併都會改變結果。對 `OPEN_PR_AUDIT_SET_MISMATCH` 查 new／closed／stale，對 `PR_AUDIT_HEAD_STALE` 核對所有 open PR，包括 defer。已關閉未合併的 PR 排除。已進 base 的更動記 integrated；較新 head 已併上游但選定 base 尚無時，把有授權的 backport 記 regenerate。只帶精確驗證過的 head，新 head 與舊 base 大幅衝突時可 defer。
- Carry 的 source 必須是對應 live PR head，non-owner review finding 可觸發 `PR_QUALITY_GATE_FAILED`。經 operator 明確要求保留且無法當 carry 的改動可記 `regenerate overlay`，audit 記 overlay 與原因。額外修補放在獨立 overlay。前一版 fork commit 每一筆都須在 manifest 計入，舊 carry SHA 另列 drop。Regenerate 的 source 只能是前一版 `origin/main` 的 commit，外部 source 用 `-`，否則觸發 `MANIFEST_COMMIT_NOT_IN_ORIGIN_MAIN`。發版後重做本機 rehearsal 時，在隔離 clone 讓 rehearsal 仍使用釘住的 previous main／manifest，避免腳本自動 fetch 後改變基準。
- 換 base 時可用 `git am -3`／rerere 重放上一版適用 patches，遇仍有 conflict marker 或未解決檔案先停。Stacked PR 取 base PR 到 child PR 的 diff；倚賴被移除 PR 的舊 patch 可從 live head 重建。避免用 union merge 拼接 test blocks；重跑各 carry 新增的 e2e 和全套 typecheck，補齊被其他 carry 改動的 test harness。新增 workspace 後重新 frozen install。合併 e2e 清單時保留雙方案例。
- 產生新 patch 時排除 `tools/maintenance/releases/`，在 manifest 用 `previous_release_records=inherit`。Rehearsal 從釘住的 `previous_origin_main` 還原舊記錄，再比對 canonical source tree。把舊 patch 錄進新 patch 會一版版放大，曾達 73 MB。[Fork PR #35](https://github.com/swear01/hapi/pull/35)
- 核對 `hapi-session-control` 的發行來源。安裝器 overlay 已於 2026-11-18 自 maintained main 移除，技能改走 Skillshare；舊 release 和 stack branch 仍有舊內容，不能因重放取回安裝器。舊 installer 的測試曾寫進真實 `~/.claude/skills`、`~/.codex/skills`；測試須同時隔離 `HOME` 和 `HAPI_HOME`。平行 CLI suites 用不同 `TMPDIR`，避免彼此刪掉 `hapi-test-config.json` 後報 `Missing isolated hub config`。

## Review 和發布證據

- 分開讀同一 SHA 的 functional CI 和 bot verdict。需求、推送及核准各自對應要發的 head；沒有 verdict、timeout、模型通道 503、Codex server-info 不足或 `Review skipped: no items were selected` 都沒有 clean review 證據。沒有 rollup checks 時另查精確 SHA 的 Actions runs，fork 的 `action_required` 可能尚未建立 job。另一個 fork PR 的 clean verdict 只適用它自己的 head。大 diff 可能讓 Swear Review 或 Bugbot 逾時，依當前 review policy 取得可用 provider 的 exact-head verdict；對 finding 用 base/head diff 確認是新引入還是原本就有。
- 2026-11-28 的例子是 [upstream #2168](https://github.com/tiann/hapi/pull/2168) 在 `d17463aa` functional CI 通過，但 [Codex review 高需求退出](https://github.com/tiann/hapi/actions/runs/45743777177/job/139654817108)。維護版 v0.36.7.1 帶了 [#2564](https://github.com/tiann/hapi/pull/2564) 的手動收合記憶，完整展開與收合修正仍 defer。四個 Chromium desktop/mobile cases 只證明該 #2168 head 的行為。下次重查 live review 與要發布的 tree。
- Fork dispatch 422 `Actions has been disabled for this repository` 即使 API 顯示 enabled 也曾真實發生。發版前用手動 workflow probe，API 重存 permissions 曾無效，需要使用者從 GitHub 恢復。Close/reopen 沒觸發時，同 SHA 臨時分支可跑 push Test，iOS/Android 等仍以 workflow 的 exact-main/path filter 為準。macOS job 在零秒取消、沒有 steps 時，看 runner image annotation；歷史上退役的 `macos-14` 需要隨 release pin 換掉。
- 發版需要隔離 rehearsal、tree equality、frozen install、完整測試、operator 核准、以及 artifact digest 和 signing 核對。只對 maintained fork main 使用明確帶有舊 SHA 的 `--force-with-lease=main:<old-sha>`。Main 先更新再推 tag，否則 release-from-main 會拒絕。核對發布的 artifact 檔名全名，`darwin-arm64` 子字串同時命中 CLI 和 Desktop。macOS 比對 designated requirement；確認 exact-main／tag runs 及 assets，才宣告相應發布階段完成。
- 測試失敗以同主機、同負載的未修改 baseline 分類。冷 dynamic import、Windows integration globalSetup hub ready、兩 CPU fixture cold render 可使 case timeout，甚至跟著印 `No test files found`。分開量 import／實際執行，保留功能 assertions 與 continuous-streaming case。跑 test 時檢查外層 `HAPI_DISABLE_PEER_TOOLS`、`HAPI_INVOKED_CWD` 洩漏，Node 的 built-in localStorage 壓過 jsdom，以及 macOS `/var` 與 `/private/var` 路徑解析。選擇各 workspace 的正式 test runner。Native 前階段失敗可能讓後續 iPad stage 未執行；同時查 skipped stages。

## Fleet handoff、DB 與 rollback

- 先核對 supervisor 和現行連線，備份 DB，逐機保存 active-session ID／PID／start time 基準。部署後檢查 exact binary、Runner state 與 supervisor PID、Hub schema／`quick_check`、實際 served Web asset，再用各目標 agent 的 model-list 或 disposable live canary。Runner online 只驗證 Runner。透過 Hub `/api/machines` 的 `metadata.happyCliVersion` 讀回版本；session 的 machine ID 在 `.metadata.machineId`。不可達主機或未做 child canary 單獨回報。
- 換 NFS binary 可令多台 Runner 自行 handoff，留下 supervisor restart loop 或短暫消失的 `runner.state.json`。先同時重讀 state 和 supervisor。確定 split ownership 時，依 supervisor runbook 只收掉已核對的 `hapi runner start-sync` state PID，確認無 `--started-by runner`，再由原 supervisor 接手。保留 systemd `KillMode=process`、launchd `AbandonProcessGroup` 或 PM2 no-treekill 等現有 child-preserving 設定；Windows 也驗 Scheduled Task 是否重新持有 runner。Retention 要從 live exe／maps／fd／cwd 驗證，NFS `.nfs*` 或只有部分 host 可查時保留。見 [supervisor 操作](../tools/hapi/supervisor-safe-operations.md)。
- `hapi-ctl.sh` 有 Mac 中心假設；在 Oracle 執行 `status mac` 曾讀到 Oracle 的本地 binary。先證明目標 host 和 SSH TCP port 可達，再派給有連線的運維機。Ping、Tailscale ping、`ssh -G` 都不證明 SSH／遠端部署成功。Windows 早年 `hub-artifact` 升級路徑已退役。Windows Codex `invalid_configuration` 先看 Scheduled Task 的啟動 script 是否將 `HAPI_CODEX_APP_SERVER_BIN` 指到已刪除的版本資料夾；一般環境沒有該變數、CLI 在 PATH 正常，都不能排除 task 內覆寫。Long-lived HAPI session 也會保留舊 PATH；先從已存在的 WinGet 位置驗證，再決定是否重裝。
- 舊版 fork schema 29 到官方 35 是單向遷移。當時真實 DB 副本 rehearsal 與正式升級均通過；正式約 6 分 20 秒，健康輪詢會先 timeout。此次需補 `idx_messages_future_scheduled_queued`，回滾用升級前 DB 備份；未來版本先對副本重驗 migration、index 和 session count。官方 v0.35.0 已移除舊 `stop-peer`／`/sessions/:id/stop`；必須用舊 CLI 和舊 Hub 先做 idle-wait＋非封存的遷移，再切 Hub。Runner `stop-session` 的 SIGTERM cleanup 會封存，不能當成等價操作。新 Hub archive endpoint 曾拒絕 inactive session；空白 row 的刪除屬另行授權。
- Process-only stop 要先送 `archive:false` RPC，取得 acknowledgment 並用所回報的 raw PID 觀察退出。Hub lease 過期、`active:false`、CLI 不可達、PID identity 不明或只看到 session ID 不能證明程序已退出；不確定時 fail closed。Terminal 已 active 的 fast path 需有 ack 之後的新 session-end。不要為了觀察而發送 SIGTERM。Reference: [session lifecycle exit confirmation](../tools/hapi/session-lifecycle-exit-confirmation.md)。

## 容易重現的實作陷阱

- Project Delete Group 的 Bun SQLite `changes` 包含 cascade，會誤觸 direct row 數量檢查。用 `DELETE … RETURNING id` 數 direct sessions；transaction 前同時檢查 DB archive／namespace 和 live SessionCache，重連狀態可能已更新在 cache 而 DB 尚未知。任一不符即整筆拒絕。Sidebar 搜尋、pinned/hidden rows 不能截斷群組成員；header keyboard handler 只處理自己，避免吞掉 Copy Path／New Session。
- Composer 的語音 send 將 live 空 draft 和 unmounted composer 分開。清掉又打相同文字屬新 draft，靠 session revision 與文字比對；drop 後需新的全域 revision，避免舊 token 再匹配。Delayed transcript delivery 前查最新 session 及 permissionMode，離線才走現有 resume，失敗保留 recovery；`useScribe` 某些版本自行在 unmount 關連線，需讓 Scribe finalization 有明確擁有者和 timeout。對 capture stale deps 的測試用穩定 assistant-ui API identity，以 production 的 `onSend()` 形式呼叫並測 attachment hydration 前後。Wide primary 和 compact toolbar 都要接 direct-send；drag handler 留在 `RichComposerInput`。Mock 證據與真實語音供應商連線分開回報。
- React 比對 `getSnapshot` 需要參考穩定。上次發生 `Maximum update depth exceeded` 時，`@assistant-ui/tap` lock drift 暴露舊 core 訂閱錯 `_core` 而非 `_stateBinding`，未連線狀態反覆配置新物件。先核 lockfile 與 audited patch，測真實快速 scroll。
- iOS 回收 fixture 用實際回收／重建訊號、保留高度和閱讀 anchor assertions。iOS 15 起 prepared-cell cache 會讓 `cellForItem(at:)` 回傳非顯示 cell，應只在測試停 prefetch，模擬拖曳通知 renderer，等待實際 cell 寬度和置中。Preferred-height invalidation 用 `prepare()` 後的 indexed frames 做 anchor compensation，避免使用舊 `originalAttributes.frame`。只延長 sleep 或放寬誤差會掩蓋錯誤。
- ACP advertised effort 可在 startup 被 provider 拒絕。報錯、發布 discovered effort，再繼續 prompt；startup 與 queued send 分開測。Cursor Auto toolbar 和 picker 需共享 flavor-aware normalization；切換模型時的內部 `/auto-review` 只應帶 permission intent，避免舊 model snapshot 在 backend restart 後覆寫 Auto。測實際 dispatch 與最終 model，不只 enqueue。
- Bun compiled child 的 `process.cwd()` 會解析 symlink。目錄 guard 仍需區分 logical path，編譯分支有明確 cwd 時用既有 `HAPI_INVOKED_CWD`，不要放寬 Hub 匹配。
- Peer send 的 intentional sends 各用新 UUID `localId`，同一 intent 的 native mirror 共用 ID；內容 hash 會吞掉合法重複訊息。Completion 以消費該 localId 和對應 ready event 配合；單靠 thinking／短時間 idle 會誤把 queued、慢啟動、被打斷或後續回合判為完成。任務離開最新頁時查 queued-state，取消會刪掉該列；分頁用 `invokedAt ?? createdAt`，注意同 timestamp 及 steer 無獨立標記。Pi 歷史版本不提供 ready，不能為它推論同樣的 completion。重疊工具 PR 與現行 contract 先核對上游。
- Codex duration 用獨立 `turn_duration` metadata，重送 token_count 會重算用量。用 live start／matching completed turn，local 用 record timestamp，remote/shared 在 async queue 前取 receipt time。持久化尾頁可缺起始 user row。關聯 native turn ID，涵蓋 Code Mode hook、late token_count 和既有無標記資料；避免舊 turn 覆寫較新 usage。Live Plan call/result IDs 同 turn 穩定、跨 turn distinct，否則會重用舊 Plan 卡片並清掉摘要。
- Pi RPC delta 已排 promise chain 時，rejection／cleanup flush 需要進同一 queue 並 await，避免掉最後一段。純 CPU 圖片註冊保持同步可去掉這個排序依賴；讀本地檔在同一 FD 做 fstat 和有界讀取，檢查增長、保證 close、避免 FIFO 阻塞。保留完整 PNG magic、error status 和 nested image payload。Pi execute signature 以 pinned tag 為準，驗證使用實際 RPC、原 extension 和隔離 model endpoint。

舊版發版清單、各機結果、精確 head 和測試收據見 [整理前固定版本](https://github.com/swear01/agent-memory/blob/d1e08b2dad393fd2ecf9dbbee4f69d53bd6d8755/projects/hapi-fork-maintenance.md)。
