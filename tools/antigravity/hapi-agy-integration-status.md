---
title: HAPI × Antigravity (agy) 整合狀態與登入前提
scope: tools/antigravity
tool: Antigravity CLI (agy)
status: active
confidence: high
evidence: >-
  2026-09-20 在 mazu/zeus 對上游 tiann/hapi 與 fork swear01/hapi 交叉查證：
  上游 PR/issue、release audit TSV、hub DB sessions metadata、runner log，
  以及 `agy --output-format=json models` 與 `agy -p` 的實際輸出。
created: 2026-09-20
updated: 2026-10-30
tags:
  - antigravity
  - agy
  - hapi
  - integration
---

## 2026-10-30 常駐 stdin transport（HAPI PR #2248）

- #2244 已於 2026-10-29 合併；傳輸重寫另開 #2248，基底 upstream `013ace8e8`。
  此處是分支實作與驗證結果，合併／release 狀態另查。
- 改為 lazy spawn `agy --input-format stream-json --output-format stream-json`，
  逐行寫入 `{event:"user",message:{content:prompt}}`。同 model/effort/permission
  設定重用 child；config key 不含後來從 init 採用的 conversation UUID。
- `result` 是 turn boundary，process close 是 transport boundary。先 flush
  planner/sendChain、派送 acknowledgement，再接受下一輪；stdout buffer 和
  listeners 屬 process，planner/acceptance/ack/model attribution 屬 turn。舊
  process 的晚到 close/error 不能清掉新 process 或終止新回合。
- 設定改變先關 stdin，給 500ms 正常 EOF 退場，再必要時 kill process tree；
  active Stop 直接 kill tree，下輪 `--conversation` resume。任何 prose/tool/
  user_input/result 證實接受後，不可自動重播該 prompt。未接受才 bounded retry。
- native 空字串／純空白會 ERROR+exit1，HAPI 必須寫入前攔截並保全原訊息。
  `/model` 等 CLI 自答命令會 ERROR+exit2；control_request/response 也不支援，
  不能把 stdin 協定當 ACP 或用 slash command 偷改 runtime 設定。
- stderr soft-deny 可在 result 之後才到；需保留 idle callback 去重顯示，不要為
  等 stderr 而阻塞下一輪。whole-kill 與 queue wakeup 相撞時，已 dequeue 的
  delivery 仍須在 kill 返回前 restore/ack，不能漏訊息或放行新程序。
- agy 1.4.11 實測：原生兩輪及 source driver 兩輪都同 process/init 一次/result
  兩次、第二輪保留 context、沒有工具呼叫。source driver 確認兩次 ready/ack，
  idle kill 後 process cleanup 完成。原生關 stdin exit0；idle SIGINT exit1，
  所以 Stop 後重啟是必要方案（未另做 active-turn 真實推論中斷）。
- num_turns、usage、duration 是 session 累計；本次不加入新的 usage 計數，將來
  接用量時不能把各 result 的累計值逐輪相加。對外 transcript 格式保持原樣。
- CLI 全套 3659 pass/16 skipped；最後 persistent regression 檔 17/17 pass，
  CLI typecheck 通過。所有 CLI test invocation 都用不同 TMPDIR。
- 最終驗收 head `01b93147d`：完整 Test CI、integration、Windows checks
  與 HAPI Bot review 全部通過。2026-10-30 再查 #2248 仍 OPEN；目前帳號只有
  upstream READ 權限，合併被拒絕，乾淨 worktree 保留待上游合併。

## 2026-10-29 模型／effort 修正實作（HAPI PR #2244）

- 實作分支以 upstream `d9acc0c31` 為基底；PR #2244 的合併／release 狀態需另查。
  Web 新建表單和 chat composer 分開選 model family 與 effort，CLI launch、resume、
  config RPC 統一正規化為 base model + effort；這個 PR 不改 per-turn transport。
- Raw machine catalog 保持原格式；shared `getAgyModelChoices` 只有同 family
  至少兩個不同 effort variants 時才合併，effort 選項取自 catalog。單 variant
  保留原 wire ID、隱藏 effort。`splitAgyModelSelection` 僅拆已知原生前綴，
  `custom-high` 等自訂名稱不拆。
- RPC 回傳實際 normalized `applied` model/effort，hub 一次保存兩個欄位；
  保留 enqueue-time snapshot。明確 effort 優先於舊 slug 的尾綴；固定模型清掉
  stale effort。catalog 不足以確認的未知 variant 衝突會明確拒絕，不默默換模型。
- 新 base IDs 不能再經 NewSession 的靜態模型白名單驗證，否則保存後下一次開表單
  就重設為 Default；agy 必須和其他 dynamic catalog agent 一樣等 live catalog 驗證。
- 真實 agy 1.4.10、live catalog 13 models，經 source normalize → build args →
  原生唯讀 `/model` 驗證四組：Argon medium + high → argon-high；base + high →
  argon-high；固定 gpt-oss-medium + stale high → 保留 medium 且不傳 effort；
  base + null → 原生 medium/default。四組 exit 0、SUCCESS、num_turns 0，不需推論。
- **不能在 bootstrap 回報之前 await 模型查詢**：catalog probe 的 timeout 和
  非 Codex runner startup webhook deadline 都是 15 秒；冷 cache 查詢卡住會讓
  runner 在 fallback 前先殺 wrapper。PR review 抓到此回歸，後續修正先 bootstrap
  回報、裝 lifecycle/kill，再 await catalog；default model 不查 catalog。
- 初始化期間沿用 ApiSessionClient.pendingMessages 保存早送訊息，正規化後才掛
  onUserMessage；RPC 等初始化完成，避免設定被初始化結果覆寫。cancel/retry 必須
  先由擁有 pending buffer 的 ApiSessionClient 移除舊項，再 fallback flavor queue
  callback；否則取消會失效、重試會重複執行。deferred lookup 和 real update-handler
  回歸測試共 59 個通過，CLI typecheck 通過。
- 冷 cache 的 explicit model 首輪仍有查詢延遲，但不再阻擋 startup webhook。
  沒有 catalog 時已 canonical 的 base + effort 可保留，無法確認的 variant 不猜
  configurable base。此次未改 per-turn transport 或 runner deadline。

## 2026-10-29 再查：model / effort 衝突與新版 stdin transport

以下以 HAPI `5d8deb201`、本機 agy `1.4.10` 原始碼和唯讀命令驗證；後面的
2026-09-20 記錄是歷史背景，尤其「只能 per-turn spawn」已不成立。ACP server
目前狀態本輪未重查，不可把舊 RC 限制當成最新結論。

- `agy -p /model --model gemini-4-argon-medium --effort high --output-format stream-json`
  在推論前直接 exit 1：`--model gemini-4-argon-medium conflicts with --effort=high`；
  result 的 `response` 為空，真正原因在 `error`，usage 全零。
- 移除 `--effort` 後同一個 `/model` 查詢成功，解析 effort 為 `medium`。
  base 名稱 `--model gemini-4-argon --effort high` 亦成功解析為
  `gemini-4-argon-high`。不要對所有名稱直接刪字尾：需由 live catalog 和 CLI
  resolution 驗證 family、variant 與可用 effort。
- `agy -p /effort --output-format json` 可取得
  `command.data={adjustable:true,current:"medium",available:["low","medium","high"]}`。
  help 已列 low/medium/high/xhigh/max，但各模型可用值應用原生查詢確認。
- HAPI `runAgy.ts` 從 launch opts 固定保存 `sessionEffort`；config RPC 只更新
  model/permission，沒有 effort handler；`agyHeadlessDriver.ts` 每輪同時傳
  model slug 和舊 effort，且只接受 low/medium/high。切 model 不會清掉舊 effort，
  resume 也會還原 stored effort，故可產生上述衝突。當前 web 新建表單的 agy
  descriptor 沒有 effort，resolvedEffort 亦排除 agy；不可直接歸因當前 web
  新建表單。CLI/runner API/resume 均是可能來源，個案來源仍須 session 證據。
- `agyNdjsonParser.ts` 丟棄 result.error 和 usage；driver 失敗時誤讀 response
  當原因。stdout 已用 setEncoding('utf8')，不能直接將字串問題歸咎 byte split。
- 官方 headless 文件與本機 help 均確認 `--input-format stream-json
  --output-format stream-json` 可用單一常駐程序處理多輪 stdin user 訊息；
  result 是回合邊界，不能再等 process close 才 finish turn。官方文件也明確說
  `/model`、control_request/control_response 會 ERROR 並結束串流，故仍需在設定
  改變時規劃重啟與 conversation resume，不能假設已有雙向控制協定。result
  usage/duration/num_turns 是 session 累計，導入時避免每輪重複計數。
- 改善順序：先統一 model/effort 的有效設定，避免矛盾參數；再整理 family 與
  effort UI、支援範圍；最後改常駐 stdin transport。此次僅調查，未修改 HAPI。

## 整合架構（HAPI 端）

- agy 是 HAPI 官方支援的 agent（`hapi agy`、web 選 Antigravity）。上游
  `tiann/hapi` PR #1320（2026-08-04 合併）先以 PTY 包 TUI，PR #1591
  （2026-08-16 合併，本機 owner 提出）改成 **headless print-mode transport**，
  刪掉約 8k LOC 的 PTY/鍵盤/畫面 scraping，設計文件在
  `docs/design/agy-headless-transport.md`（以 release tag 為準）。
- 每回合固定 spawn 一次：
  `agy -p <msg> --output-format stream-json --conversation <uuid>
  --model <slug> --mode accept-edits|plan --effort low|medium|high
  --print-timeout 30m [--dangerously-skip-permissions]`，解析 NDJSON 的
  init / step_update / result；conversation id 取自 init envelope，後續回合用
  `--conversation` resume。
- **agy 只能 remote mode**：hub 的 spawn 路由對 `startingMode !== 'remote'`
  回 400，因此沒有 terminal 檢視、沒有本機 PTY 模式。
- 模型清單來自 `agy --output-format=json models`（含文字表格與硬編鏡像的
  fallback）；in-session 切換模型會讀同一份 machine catalog（#1814）。
- Session 標題從 `~/.gemini/antigravity-cli/conversation_summaries.db` 同步
  （#1476）；spawn 時會濾掉 `SSH_CLIENT`/`SSH_CONNECTION`/`SSH_TTY`。

## Fork（swear01/hapi）政策

- #1320 在 audit ledger 一直是 `drop`，但那只表示「不重放 carry patch」；
  官方 base 自 v0.26.0.1 起已含它，fork build 因此一直有 agy 支援。
- v0.30.7.1 之前的 audit 都把兩個 agy 修復 PR 列為 `defer`：upstream #1629
  （delta UTF-8 破損造成長非 ASCII 答案重複送兩次，仍有 HAPI Bot findings）、
  #1642（失敗回合把答案當錯誤理由，應讀 `error` 欄位）。目前 fork 不含這兩個
  修復，且上游也未合併。

## ACP：agy CLI 沒有，但 Google 另有官方 ACP server

- **agy CLI 本身沒有 ACP**：`agy --help` 沒有 `--acp` / `agent acp`，binary 沒有
  `agent-client-protocol` 字串，changelog 到 1.2.7 也無 ACP；上游 feature request
  #31 / #195 / #704 仍 open（社群指出 Gemini CLI 有 ACP、agy 沒有，block/buzz
  因此無法支援 agy）。因此 HAPI 只能走 print-mode NDJSON。
- **但 Google 另有官方獨立的 ACP server**：ACP registry id `antigravity-acp`，
  執行檔 `agy_acp_server.par`（建於 Antigravity Python SDK），Zed 的 ACP registry
  頁面有列；平台為 linux-x64/arm64、win-x64/arm64、mac-arm64（darwin-x64 未發佈，
  issue #951）；registry 1.1.1，build tag `agy_acp_server_20260818_01_RC01`，支援
  initialize / authenticate / session/new / session/prompt / session/load /
  session/close / session/set_config_option。
- **HAPI 完全沒接**：repo 內 0 筆 `agy_acp_server` / antigravity+acp 參照，agy 仍是
  print-mode；HAPI 已有的通用 ACP backend（`cli/src/agent/backends/acp/`）目前只用於
  Cursor / Grok / Copilot / Kimi / OpenCode / DSH。
- RC 已知缺陷（多為 open issue）：checkpoint 後重啟 session 無法恢復、live update
  缺 diff、session/prompt 不回、Windows 每次 spawn 冷啟約 16s、不報 token
  usage/quota、dotted MCP 名稱被拒。
- 同生態已有先例：pi 的 `pi-antigravity-acp-provider` /
  `@estebanforge/pi-antigravity-bridge` 把 `agy_acp_server` 當第二個 turn engine
  （stream-json 仍是預設）。踩雷：ACP binary 不吃 model flags、RC01 的
  kill+reload 失敗、無 usage 回報。
- 對 HAPI 的含義：要讓 agy 走 ACP，需新增 ACP launcher，並處理
  `agy_acp_server.par` 的取得、認證與模型選擇（ACP 端目前沒有 model flag）；
  在那之前 print-mode 是唯一可行路徑，也解釋了沒有雙向 permission 審批、
  plan/todo、question UI、ACP model catalog、`session/load`、usage 計量的原因。

## MCP：agy 原生支援，HAPI 不注入

- agy CLI 自己有 MCP 管理：`agy mcp add/remove/list/enable/disable`（stdio 與
  HTTP）。**CLI 讀的是 `~/.gemini/config/mcp_config.json`**（或 workspace
  `<workspace>/.agents/mcp_config.json`）；`~/.gemini/antigravity/mcp_config.json`
  是 Antigravity IDE 的檔案，CLI 不讀（本機該檔有 context7，但 `agy mcp list`
  仍顯示 `No MCP servers configured`，因為 CLI 檔是 0 bytes）。
- HAPI **不會**把 HAPI 自己的 MCP bridge 注入 agy session（upstream
  `docs/guide/agents.md` 明講 "no HAPI injection"）。因此 agy session 沒有
  `hapi_change_title`、`display_image/video/media`、`skill_lookup`、
  `ping_peer`/`inspect_peer`/`spawn_peer` 這些其他 flavor 有的工具；標題改由
  agy 原生 conversation title 同步（#1476），session 控制改用 fork carry 的
  `hapi-session-runtime` skill（安裝在
  `~/.gemini/antigravity-cli/skills/hapi-session-runtime`）。
- agy 的權限沒有 hook bridge：用 agy 自己的 `settings.json` allow/deny 規則
  （`request-review`）或 `--dangerously-skip-permissions`
  （`always-proceed`）；沒有 allow rule 的 tool call（含 MCP）會被 agy 自動
  拒絕並在 chat 顯示提示。
- 要在 agy + HAPI 用 MCP：把 server 寫進共用 home 的
  `~/.gemini/config/mcp_config.json`（一次覆蓋所有 Linux runner）或 workspace 的
  `.agents/mcp_config.json`，並補 allow rule。遠端不能打 TUI 的 `/mcp`。

## 機隊操作前提：agy 必須在共用 NFS home 登入

- Linux runner 的 `$HOME` 是 NFS 共享（`192.168.1.200:/volume1/nfs-home`）。
  `~/.gemini/antigravity-cli` 與 token 檔
  `~/.gemini/antigravity-cli/antigravity-oauth-token` 都在這個 home 內，
  所以**登入一次即可覆蓋所有掛同一 home 的 Linux runner**。
- 2026-09-20 實測 mazu 與 zeus 都未登入：`agy --output-format=json models`
  回 `Please sign in to view available models`，`agy -p "..."` 直接印出
  Google OAuth URL 並等待 60 秒，token 檔不存在。
- 未登入時 HAPI 端不會報錯給使用者：runner log 只看得到
  `[agy-headless] Starting headless driver`、`List Agy models request` 與
  cleanup，hub DB 會留下 0 筆 message 的 agy session（mazu 2026-09-20 三筆
  即此情況）。反之，Mac（459 messages，2026-09-19~20）與 oracle
  （192 messages，2026-09-11）有真實對話，代表那兩台已登入。
- 驗證真實登入狀態不要只看 log 的 `You are not logged into Antigravity`
  （啟動競態會誤報），要用功能驗證：`agy -p "reply only with OK"` 或
  `agy --output-format=json models`。

## 快速查詢

```bash
# 每個 flavor 的 session 數（hub DB，metadata 是 JSON）
python3 -c "import sqlite3,json,collections;c=sqlite3.connect('file:<hub-home>/hapi.db?mode=ro',uri=True);print(collections.Counter(json.loads(m or '{}').get('flavor') for _,m in c.execute('select id,metadata from sessions')))"

# runner log 中的 agy 活動
grep -E '\[agy|Agy' "<runner-home>/logs/"*runner.log
```

真實 agy session 的特徵：`metadata.flavor == 'agy'`、messages 表有實際
rows、runner log 有 `[agy-headless]` 的 main loop 與 cleanup；0 rows 要懷疑
登入或模型探測失敗。
