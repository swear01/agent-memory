---
title: DeepSeek Latch Gateway fleet routing and automatic failback
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: Bun/systemd/launchd/Windows-Task-Scheduler
status: active
created: 2026-08-25
updated: 2026-09-23
tags: [gateway, opencode-go, routing, failback, hapi, swear-review, model-alias, openrouter]
---

# 架構

Gateway 是分散式部署，不是中央單點：每個 client 都呼叫同機 loopback 上的
gateway process。現行 fleet 共八台，Mac、mazu、athena、cthulhu、valkyrie、Oracle、swop 使用
`35001`；Zeus 因為同機另一帳號會占用 `35001`，所以使用 `35002`。

`routing.yaml` 只定義 model 到 priority group 的資料；gateway binary 內的
`PriorityLatchManager` 才負責 fallback、cooldown、half-open probe 與自動
failback。因此 routing code 修正必須部署 binary，只改 route file 不足以生效。

# 部署不變量

- Mac 使用獨立 arm64 binary 與 LaunchAgent。
- mazu、athena、cthulhu、valkyrie 透過 NFS home 共用同一份 x64 binary，
  但每台有自己的 process，仍須逐台 restart。
- Oracle 使用獨立 arm64 binary 與 systemd user service。
- Zeus 使用獨立 x64 binary 與 systemd user service。
- swop 使用 Windows x64 binary，正式位置為 `<program-data>/DeepSeekGateway/deepseek-gateway.exe`，由 SYSTEM Scheduled Task `DeepSeek Gateway (SWOP)` 管理；使用者目錄 `.local/bin/deepseek-gateway.exe` 副本也須同步。
- Gateway 清單不能從舊 HAPI／Linux 清單推測；必須包含 swop 並核對真實 supervisor。
- 更新 Swear Review 的 caller header 不代表 Gateway 已更新；兩者是不同部署。
- 先在 live path 之外完成 staging 並核對 SHA，再以 atomic replacement
  換入正式路徑、保留已知可用備份，最後 restart。
- 文件與記憶不得保存 API key 值。

# Recovery 行為

Quota failure 會先依序耗盡三個 OpenCode Go account，再進入低優先級 route。
Cooldown 從 1.5 小時開始，failed recovery probe 後倍增，最高 24 小時。
到期後由一個真實 request 擁有 half-open probe；其他 concurrent request
繼續使用 fallback。Probe 成功便恢復到可用的最高 priority。

# 2026-09-15 OpenCode key 輪替與槽位對調

使用者要求淘汰舊第二把 key，並將替換後的新第二把與原第三把對調。
最終 `OPENCODE_API_KEY_1` 保留原值、`OPENCODE_API_KEY_2` 使用原第三把、
`OPENCODE_API_KEY_3` 使用本次新增 workspace 的 key。endpoint 的 account 編號
現在表示槽位，不能再依舊帳號名稱推定其持有的 key。

- 已完成並驗證 **7／8 台**：Mac、mazu、athena、cthulhu、valkyrie、Zeus、Oracle。
  swop 尚未完成本次 key 輪替；最後已知 SSH 位址不可達，恢復連線後需補做，
  不能把七台成功當作整個 fleet 完成。
- 憑證來源為 `<remote-home>/.secrets`；已存在的
  `<remote-home>/.dsh/.credentials.yaml` 同步第二、第三槽，缺少第三槽時補齊。
  第一把及其他 provider 憑證保持不變；key 不寫入 gateway config、文件或記憶。
- 七台均核對磁碟與運行中程序的三把 key 對應關係，並實際透過
  `opencode-go-3` 推論取得 HTTP 200 與 `OK`。本次證據包含 OpenCode 上游成功，
  不只是 Command Code fallback；不代表未來配額或所有模型永遠可用。
- Mac、mazu、Zeus、Oracle 等 gateway established 連線清空後重啟 gateway，
  HAPI session roots 分別保留 5／5、1／1、2／2、12／12。沒有重啟 HAPI Runner。
  Athena、Cthulhu、Valkyrie 恢復連線時，NFS 憑證及既有 gateway 程序均已載入
  新順序，因此只驗證、不再重啟；當時三台沒有 runner-started session roots。

驗證陷阱：`POST /switch?index=2` 只切換 `routing.yaml` 的第一個 model route，
各機第一條可能是 `deepseek-flash` 或 `deepseek-v4-flash`。測試必須使用該
route 的 model，核對 `X-Gateway-Active-Endpoint: opencode-go-3`，完成後恢復
測試前的 index。單看 HTTP 200 可能其實仍在使用第一把 key。只更新
`.secrets` 也不等於程序已重載；先看 runtime，再決定是否需要 gateway restart。

# 歷史 rollout（七台，非現行清單）

2026-08-25 已對 commit `0944c14` 完成 35 tests、typecheck、三平台 build，
並部署至七個 gateway。Live check 驗證 service health、各平台 binary SHA、
Priority 1 route 載入與第三個 OpenCode endpoint 存在。Oracle 原本已是相同
arm64 build，因此沒有做多餘 restart。正式環境沒有刻意耗盡 quota 等待
1.5 小時；自動 failback 時序由 unit 與 integration tests 驗證。

再次使用這份紀錄前，仍須重查 live hash、service、config 與 endpoint status。

# 2026-09-07 八台 session header rollout

PR #10 合併提交 `73f0fb5`（review head `ae9963c`）保留既有 priority routing，
修正共用 forwarder 的 OpenCode POST header。46 tests、typecheck、四平台
建置通過；Swear Review 對該 head 為 0 findings。八台 binary、程序及 health
均已驗證，包括最初漏掉、後續補部署的 swop；config/routing 保持不變。

- 保留 caller 的 `x-opencode-session`，優先於 endpoint 靜態 header。
- 缺少時依序接受 `x-deepseek-harness-session-id`、`x-session-id`、`x-session-affinity`、`session_id`。
- 完全沒有 ID 時，以 client identity 與開場 user message 派生 SHA-256 affinity；重試及保留開場的後續對話保持相同值。
- 相同 client 與相同開場會共用 affinity；compaction 更換開場會改變它。精確隔離需要 caller 傳入穩定 ID，不能把雜湊 fallback 說成可靠 conversation identity。
- 同時沒有 ID 與開場時，OpenCode 路徑回傳本地 HTTP 400 `missing_session_id`；其他 provider 不變。

隔離的原生封包測試驗證 header、alias、重試及多輪穩定性。八台真實推論
皆 HTTP 200，但實際走 CommandCode fallback；這證明 gateway inference
可用，不證明 OpenCode upstream inference 成功。不可混用兩種驗證證據。

Linux 應比對正式 binary 與 `/proc/<pid>/exe` 的 SHA-256；Mac、Windows
則核對新程序的 executable path、啟動時間與已安裝 binary hash。保留舊版
備份，等 established 連線清空後只重啟 Gateway，不重啟 HAPI Runner。

# 2026-09-11 model id 已別名到 V4.1 Flash

DeepSeek 於 2026-09-10 發佈 V4.1-Flash（552B MoE、Causal Encoder–Decoder，input 8B /
output 16B active，原生多模態）。官方 changelog 明確寫：canonical id 改為
`deepseek-flash`；V4-Flash 與 V4-Flash-Vision-Exp **已 retired**，`deepseek-v4-flash` 與
`deepseek-v4-flash-vision-exp` 只是**暫時 route 到 V4.1 Flash** 的相容別名；2026-09-14
04:00 UTC 起至 V4.1-Pro 發佈前，`deepseek-v4-pro` 也 route 到 V4.1 Flash 並以 Flash 價計費。

因此設定裡仍寫 `deepseek-v4-flash` 的 client，現在拿到的是 V4.1 Flash，不是 V4 Flash。
以 modality budget 指紋實測（方法見 `domains/llm-inference/model-identity-verification.md`）：
同一張圖送 gateway 路徑與 Command Code 的 `deepseek/deepseek-v4-flash` 都得
`prompt_tokens=235` 且正確讀出圖中數字，與 `deepseek/deepseek-v4.1-flash` 一致；
純文字的 `deepseek/deepseek-v4-flash-fast` 只有 18 且回 "I cannot see an image."。
舊 V4-Flash 是純文字模型，所以能讀圖就已證明 id 被別名。

**不可用模型自述判斷。** 同路徑問 model/version 時 `content` 為空，reasoning 抓到
system prompt 的日期幻覺（自稱 current date `2026-05-07`）再自行推論出 GPT-4 時代的
knowledge cutoff 後迴避作答。

## 當時狀態與待辦（swever，2026-09-11）

- 三個 OpenCode Go account 全部無額度：account 1 `CreditsError Insufficient balance`；
  account 2 `GoUsageLimitError`（3 天後重置）；account 3 `GoUsageLimitError`（8 天後重置）。
  多數 `deepseek-v4-flash` request 會落到 Command Code fallback，但 **不是全部**。
- `CreditsError` 是 OpenCode Zen **balance** 帳單，不是 Command Code 額度，也不是 Go 週配額。
  官方 Go 在週/月/rolling limit 之後若開了 Use balance，會改打 Zen credits；credits 見底就回
  HTTP `401` `{"type":"CreditsError","message":"Insufficient balance..."}`，billing URL 指向
  `opencode.ai/workspace/<workspace>/billing`。Command Code（`api.commandcode.ai`）是另一套帳。
- 舊 binary 的 `isRateLimitOrQuotaError` 只認 `429`/`402` 與 `GoUsageLimitError` /
  `weekly usage limit` 等字串，**不認** `401` + `CreditsError` / `Insufficient balance`。
  Account 1 的 401 因此不走 1.5 小時 quota cooldown，recovery probe 把它當 network
  failure（上限 15 分鐘）再探，並把原始 401 轉發給 HAPI/Pi。Live（oracle，2026-09-11）：
  active endpoint 已是 Command Code（約 1800 次成功），但 `lastSwitchReason` 反覆出現
  Account 1 → Account 2 的 `Status 401 CreditsError`。
- 源碼已補 `creditserror` 與 `insufficient balance`（不把所有 401 當額度，以免誤傷真
  auth 失敗）。2026-09-11 已把合併提交 `4fa6687`（PR #12，fix head `f4d7eb3`）部署到
  Mac、mazu、athena、cthulhu、valkyrie、oracle、zeus。七台真實推論 HTTP 200，同一
  request 第 4 次 attempt 落到 Command Code；Linux 核對 `/proc/<pid>/exe` SHA 與磁碟
  相符。swop 當日不可達（HAPI machine 列表無此機、區網 SSH 無 banner），Windows exe
  已建置但未安裝。
- HAPI 在 oracle 上這條路徑是 `hapi pi --model opencode-go/deepseek-v4-flash`，
  Pi `models.json` 的 `opencode-go.baseUrl` 為 loopback `:35001/v1`、`apiKey` 為
  `local-gateway`。不是 OpenCode CLI 直連 Zen。
- 已解決（2026-09-11 複驗，2026-09-12 校正）：先前 `routing.yaml` 沒有 canonical id、`config.yaml` 的
  `models.allow` 只有舊 id、Pi 端三層 allowlist 也只放行 `deepseek-v4-flash`。現在
gateway 兩個檔都已補 `deepseek-flash` / `deepseek-pro`（`models.allow` 共四個 id），
  Pi 端則**只把兩個 DeepSeek id 改名**，其餘 allow 條目保留：`enabledModels` 與
  `model-filter.json` 各 8 個 model（Codex 四個 ＋ `meta/muse-spark-1.2-contributor` ＋
  `valkyrie-ninfer/qwen3.8-27b` ＋ 兩個新 DeepSeek id），`strict-model-allowlist.ts` 的
  `allowed` 為 7 筆、`includes` 保留 `meta/muse-spark-1.2-contributor` 特例。
  2026-09-11 曾把這三層錯砍到只剩兩個 DeepSeek id（連 `includes` 的 meta 特例也刪），
  2026-09-12 在 swever 已還原；備份來源與還原指令見
  `tools/pi/deepseek-model-id-migration.md`。gateway process 啟動時間晚於 config mtime，
  證明已重載。舊 route 是**刻意保留**
  給尚未改名的 client，其 fallback 仍寫 `upstream_model: deepseek/deepseek-v4-flash`，不是漏改。
- 仍未解除：`deepseek-v4-flash` 只是官方**暫時**別名，被移除後舊 route 的 fallback 會壞；
  開著舊 route 的機器要記得一起收掉或改指 `deepseek/deepseek-v4.1-flash`。
- 仍未解除：`deepseek-flash` **不在** Pi 取得的 upstream catalog 裡（2026-09-11 抓到的
  `opencode-go` catalog 只有 `deepseek-v4-flash`、`deepseek-v4-flash-vision-exp`、
  `deepseek-v4-pro`、`deepseek-v4.1-flash`），但 OpenCode Zen 的 live `/v1/models`
  仍有 `deepseek-flash`。所以 `models.json` 的手寫 override 是**必要**的，不能因為
  picker 顯示得出來就以為 catalog 裡有這筆。
- 仍未解除：`deepseek-pro` 是 local alias，不是任何 provider catalog 的 id；路由把它指到
  Command Code 的 `deepseek/deepseek-v4-pro`。2026-09-14 04:00 UTC 起官方把
  `deepseek-v4-pro` 也 route 到 V4.1 Flash 並改以 Flash 價計費，屆時這條 route 的
  upstream id 不變，但實際模型與價格都會變。
- 價格 metadata 待修：`<remote-home>/.pi/agent/models.json` 的 `deepseek-flash` 寫
  `input 0.22 / output 0.66 / cacheRead 0.007`，那是 V4-Flash-Vision-Exp 在 2026-09-10
  降價**前**的價；V4.1-Flash 現價是 `0.15 / 0.60 / 0.003`（Pi catalog 的
  `deepseek-v4.1-flash` 與 `deepseek-v4-flash` 一致，Command Code 頁面 off-peak 亦同）。
  改名時只換 id 沒換 cost，pi 的成本顯示會高估約 47%。同檔 `deepseek-pro` 的
  `0.66 / 1.98 / 0.022` 目前正確。
- OpenCode upstream 的 catalog 仍把 `deepseek-v4-flash` 標成 `input:["text"]`，只有
  `deepseek-flash` 標多模態。額度用盡無法驗證 OpenCode Go 這條路徑是否也已別名；
  額度重置後須重驗。歷史上此 id 曾名不符實（opencode issue #40409：`deepseek-v4-flash`
  自稱 V3.2、knowledge cutoff 2025-05，該 issue 已關閉），不能假設 OpenCode 端行為與
  Command Code 一致。

## 2026-09-11 client 端 id 改名（swever 已完成）

Gateway route 補上 canonical id 後，還要同步每個 client 的 allowlist，否則 request
根本送不出來。swever 上要改的檔案與不變量：

- Pi `<remote-home>/.pi/agent/models.json`：`opencode-go.models` 只留 `deepseek-flash`
  與 `deepseek-pro`（`baseUrl` 保持 loopback `:35001/v1`）。
- Pi `extensions/strict-model-allowlist.ts`：**只把 `allowed` 裡兩個 DeepSeek id 換名**
  （`opencode-go/deepseek-v4-pro` → `opencode-go/deepseek-pro`、
  `opencode-go/deepseek-v4-flash` → `opencode-go/deepseek-flash`），其餘六筆 id 與
  `includes` 的 `meta` / `muse-spark-1.2-contributor` 特例**必須原樣保留**。這個允許清單
  同時 patch `ModelRegistry.prototype`、`ModelRuntime.prototype`（CLI `--model` 解析在
  `session_start` 之前）與 `getAuth`，所以少改一處就會出現「列得出來但送不出去」或反過來的情況。
- Pi `model-filter.json`（`defaultAction: block`）：改名是把 opencode-go 那條 rule 的
  `ids` 換成 `deepseek-flash` / `deepseek-pro`；`openai-codex`、`meta`、`valkyrie-ninfer`
  三條 allow rule 不能刪。`settings.json` 的 `enabledModels` 是**八筆**（兩個新 DeepSeek id
  ＋ Codex 四筆 ＋ `meta/muse-spark-1.2-contributor` ＋ `valkyrie-ninfer/qwen3.8-27b`），
  `defaultProvider` / `defaultModel` / `defaultThinkingLevel` 沿用備份原值
  （`valkyrie-ninfer` / `qwen3.8-27b` / `max`）。三層要一起改；只改其中一處會被另一層過濾掉。
- OpenCode CLI `<remote-home>/.config/opencode/opencode.jsonc`：`model`、`small_model`
  與 `provider["opencode-go"].whitelist`。
- dsh `<remote-home>/.dsh/settings.yaml`：`llm-deepseek.models` 是 advisory catalog
  （`@deepseek-ai/dsh-llm-deepseek` 的 schemastery `catalogModel`，必填只有 `id`，可選
  `name`/`contextWindow`/`maxTokens`），預設值是**已被 retire 的** `deepseek-v4-flash`
  / `deepseek-v4-pro`，所以要明寫新 id；`baseURL` 與 `apiKeyEnv` 不動。

## 2026-09-11 回歸：改名不得砍掉其他模型（Zeus 已修復）

有人把改名操作誤做成「白名單縮到只剩 `opencode-go/deepseek-flash` / `deepseek-pro`」，
把 Codex 四筆、`meta/muse-spark-1.2-contributor`、`valkyrie-ninfer/qwen3.8-27b` 全部刪掉。
**改名（rename）與縮減（narrow）是兩件事**：原始需求是只換兩個 DeepSeek id，其他模型必須
保留。Zeus 上受影響的三個檔案與還原來源：

- `~/.pi/agent/settings.json`、`~/.pi/agent/model-filter.json`、
  `~/.pi/agent/extensions/strict-model-allowlist.ts` 皆由
  `~/.pi/agent/*.backup-deepseek-rename-20260911-193728`（真正的改名前備份，mtime 19:38，
  id 仍是 `deepseek-v4-*`）整檔還原後只做 id 字串替換。先確認備份與 live 的非目標欄位完全
  相同，再整檔安裝，可避免手抄清單漏筆。
- 還原後 `ctx.modelRegistry.getAll()` dump 共 **8 筆**：`opencode-go/deepseek-flash`、
  `opencode-go/deepseek-pro`、`openai-codex/gpt-5.6-luna`、`openai-codex/gpt-5.6-sol`、
  `openai-codex/gpt-5.6-terra`、`openai-codex/gpt-daybreak-blue-latest`、
  `meta/muse-spark-1.2-contributor`、`valkyrie-ninfer/qwen3.8-27b`。
- 備份目錄陷阱：`<gateway-config-dir>/backup-<ts>-pre-deepseek-rename/` 這個名字**不可信**。
  Zeus `backup-20260911-195056-pre-deepseek-rename/` 裡的 `pi-agent-settings.json` 與
  `pi-agent-model-filter.json` 已經是縮減後的狀態（`enabledModels` 只剩兩筆），只有
  `pi-agent-extensions-strict-model-allowlist.ts` 還留著七筆。真正的改名前快照在
  `~/.pi/agent/` 的 `*.backup-deepseek-rename-<timestamp>`。還原前先比對 mtime 與內容，
  不要只信目錄名。

驗證不能只看檔案內容：用 `pi -e <extension>` 在 `session_start` dump
`ctx.modelRegistry.getAll()`，確認輸出的 provider/id 清單就是預期那幾筆，才證明三層
過濾同步。Gateway 端則對每個新 id 各打一次 `chat/completions`，看 HTTP code 與回應的
上游 `model` 欄位；同日 swever 上 `deepseek-flash` 的回應 `model` 是
`deepseek/deepseek-v4.1-flash`（OpenCode Go 額度用盡，實際走 Command Code fallback），
舊 id 仍可路由，符合「新 id 為主、舊 route 留 fallback」的目標。

# 2026-09-11 Zeus：gateway 端新增 alias route 並切換 client id

Zeus（port `35002`）與 swever 都已完成各自機器的改動；其餘機器仍使用舊 id，逐台 rollout。

- `config.yaml` 的 `models.allow` 同時允許 `deepseek-v4-flash`、`deepseek-v4-pro`、
  `deepseek-flash`、`deepseek-pro`。
- `routing.yaml` 新增 `deepseek-flash`（priority 1 = opencode-go-1/2/3 latch；priority 2 =
  command-code，`upstream_model: deepseek/deepseek-v4.1-flash`）與 `deepseek-pro`
  （priority 1 = command-code，`upstream_model: deepseek/deepseek-v4-pro`）；舊的
  `deepseek-v4-flash` / `deepseek-v4-pro` route 保留當 fallback。
- 只改 route file：gateway binary 與 `server.port` 不變，重啟用
  `systemctl --user restart deepseek-gateway`，不重啟 HAPI runner。改前後都用
  `python3 -c "import yaml;yaml.safe_load(open(...))"` 驗兩支 yaml。
- Zeus 三組 OpenCode Go 帳號當時都因 monthly usage limit 回 429，所以兩個新 id 的 HTTP 200
  實際都走 priority 2 Command Code；回應的 `model` 欄位分別是
  `deepseek/deepseek-v4.1-flash` 與 `deepseek/deepseek-v4-pro`。走 fallback 的成功不能當成
  OpenCode upstream 成功的證據。
- Zeus 另有 `<remote-home>/.local/bin/pi-safe` wrapper，`--list-models` 會把輸出過濾成
  opencode-go 的兩個新 id。
- 備份目錄慣例：`<gateway-config-dir>/backup-<YYYYMMDD-HHMMSS>-pre-deepseek-rename/`，內含
  `MANIFEST.txt` 記錄備份檔名對應的原始路徑（多個同名 `settings.json` 要加機器與工具前綴）。
  但這個目錄名只是命名，**不代表裡面真的是改名前狀態**（Zeus 那份已含 2026-09-11 的
  回歸縮減，見上方同名章節）；真正 pre-rename 快照是 client 端的
  `*.backup-deepseek-rename-<timestamp>`，還原前先比對 mtime 與內容。

# 2026-09-12 exhaust-route + OpenRouter on deepseek-flash

PR #14 合併提交 `9507d17`（fix head `9addc3d`）修正「priority 被當成額度牆」：
同一 request 必須依序走完該 route 剩餘 source，circuit 全開時 last-resort
再打一次。另一個 request 若正擁有 half-open recovery probe，concurrent
traffic 不得 bypass 到正在 probe 的 group 0。`extra_body` 現在會轉給上游，
但 `model` / `response_format` 仍由 route remap 與 compat strip 決定。
全 source 都是 network failure 時回 502 `upstream_unreachable`，不能因為
`maxRetries > routeSize` 就改報 429。

live routing 變更（binary-only 不夠）：

- `deepseek-flash` 補 priority 3 `openrouter-fallback`，
  `upstream_model: deepseek/deepseek-v4.1-flash`。
- 舊 id `deepseek-v4-flash` 的 OpenRouter 仍用 `deepseek/deepseek-v4-flash-0731`。
- OpenRouter `extra_body.provider.max_price` 改為 prompt `0.15` / completion `0.60`
  （V4.1 Flash 官價）；舊 cap `0.08` / `0.18` 會擋掉所有 V4.1 provider。
- Mac 原本就有 OpenRouter endpoint；NFS 四台、Oracle、Zeus 的 `config.yaml`
  先前沒有這個 endpoint。各機 systemd/launchd process env 已有
  `OPENROUTER_API_KEY`，所以新增 endpoint 時用 `${OPENROUTER_API_KEY}`，
  不要把 key 寫進檔案或 memory。

2026-09-12 已部署至 Mac、mazu、athena、cthulhu、valkyrie、oracle、zeus。
55 tests、typecheck、四平台建置；Linux 磁碟 SHA 與 `/proc/<pid>/exe` 相符，
Mac 為新 PID + 簽章後 SHA。真實 `deepseek-flash` 推論皆 HTTP 200、
`X-Gateway-Active-Endpoint: command-code`。Go 月額仍盡，這證明 failover
到 Command Code，不證明 OpenCode 成功，也不證明 OpenRouter 被打到
（CC 成功所以沒有走到第三組）。不要重啟 HAPI Runner。
部署紀錄 PR #15 已合進 `caa23fd`。

舊 binary **完全不 parse `extra_body`**。Mac live config 裡即使早已寫了
OpenRouter `max_price`，PR #14 之前也不會送到上游；只改 yaml、不換 binary，
OpenRouter 仍等於沒掛。NFS／Oracle／Zeus 連 endpoint 都沒有，即使 process
env 已有 `OPENROUTER_API_KEY`。

`/status` 的 `circuitState: open` 看的是 `consecutiveFailures > 0`，不是
`blockedUntil`。cooldown 到期後畫面仍可能顯示 open；要以 `blockedUntil`
對現在時間，或看下一筆 request 會不會 probe。

swop 仍未部署：HAPI machine 列表無此機；Mac 不在舊 `192.168.1.0/24`，
已知 Windows OpenSSH `192.168.1.206` 與 mDNS `Swear01_PC` 無回應。
Windows exe 已建置，SHA-256
`e5b960e10bae928ab12b0de957fc9da334d3df9fa898947ac85a0107198c40b7`。


## 2026-09-23：應用程式必須沿用使用者的 local gateway

使用者明確更正 AIsimpV：DeepSeek V4.1 Flash 應走 same-machine local gateway，不能因環境存在 `DEEPSEEK_API_KEY` 就改用官方 API。mazu 的 live Pi `opencode-go` 路徑為 `http://127.0.0.1:35001/v1`、公開 placeholder `local-gateway`，model `deepseek-flash`；gateway 持有實際 upstream keys。官方 API 的 402／balance=false 與這條路徑不是同一帳戶，不能推論使用者的標準 DeepSeek 開發路徑無額度。

新 client 應記錄 response 的 `X-Gateway-Active-Endpoint`、`X-Gateway-Attempt`，並帶 per-run 穩定 `x-opencode-session`；非 streaming 的 returned model 也須保存，不能只看 client model id。HTTP error 也要保存 routing metadata。Gateway 可 strip response_format／改寫 model ID／正規化 reasoning；client 的原始 JSON 不等於所有 upstream wire bytes。

同日 AIsimpV 的 urllib smoke 透過 gateway 得 HTTP 403 / Cloudflare 1010，active endpoint 是 `opencode-go-1`、attempt 1；該 client signature 被上游拒絕，未有 generation 結果。這只是當時此 client 的實測，不代表所有 client／gateway routes 都壞了，也不可倒推成官方 key 餘額問題。未修改 gateway service、keys 或 routing。

## 2026-09-23：最新 binary 仍不會對一般 403/1010 failover

- mazu 磁碟 binary 與 running executable 的 SHA-256 都是 `37bf5804cd519d836575ad76fa2304a150034df9dc6fe87fa346fda52bb7d803`，符合 PR #14 的最新 Linux x64 部署；main `caa23fd` 相較 runtime merge `9507d17` 只改部署文件。這次問題不是漏更新。Repository 沒有 GitHub Release，不能靠 package 的固定 `1.0.0` 判斷版本。
- `src/proxy.ts` 的 priority 路徑只對 quota classifier 命中或 recovery probe 的非成功回應繼續選來源；一般 active-endpoint 的 Cloudflare 403/1010 直接返回，不記錄 failure、不 cooldown。下一個 request 仍會打相同來源。README 的「走完 route」不能解讀為任何 HTTP error 都會 failover。
- 用正式 binary 加隔離 loopback upstream、dummy key 重現：連續兩次模擬 403/1010 均 attempt 1、fallback 0 hits、circuit closed；429 對照則 attempt 2、fallback HTTP 200。這是 transport simulation，不是真實模型成功；沒有改正式服務或 routing。
- 同一測試確認 forwarder 保留 urllib 的 `User-Agent: Python-urllib/3.14`。此 header 是 Cloudflare signature rejection 的候選原因，**未證明是實際唯一觸發因素**。處理時要區分 upstream access filtering 與 quota，不能把所有 403 當額度耗盡；重裝相同 binary 不會解決已重現的分類缺口。

# 2026-09-23 Cloudflare Error 1010 封鎖與 User-Agent 正規化（PR #16 / #17）

## 原因分析
- OpenCode Go（`https://opencode.ai/zen/go/v1`）前端 Cloudflare WAF 會在 API key 驗證前，直接對帶有 `Python-urllib`（如 AIsimpV / Python caller）之 User-Agent 回應 HTTP 403 / Error 1010。
- 舊版 gateway 僅單純轉發 client 端 headers，且 403 未納入 failover 判斷，導致 request 卡死在 `opencode-go-1`。

## 修正實作（PR #16 / commit `9861158`）
1. **Outbound User-Agent 預設與正規化**：預設帶上 `User-Agent: aisimpv-gateway/1.0`；當 client UA 為空或為 generic 函式庫簽章（如 `python-urllib/*`、`python-requests/*` 等）時予以替換，但保留自定義 agent UA 與 endpoint 設定之 `extraHeaders`。
2. **Session Identifier 轉發**：保留並正規化 `x-opencode-session`，支援 `x-conversation-id` 與 `conversation_id` 轉譯。
3. **JSON Body Content-Type 保證**：對帶有 parsed JSON body 之請求自動補齊 `Content-Type: application/json`。
4. **Cloudflare 1010 專用 Failover**：精確識別 Cloudflare Error 1010 / WAF 封鎖特徵，將其視為 endpoint failure 觸發暫時 cooldown（基礎 30 秒 backoff）並立即 failover 至下一組 endpoint；一般權限 403 則直接透傳，不誤判為 failover。

## Fleet 部署與驗證
- **已部署 7／8 台**：Mac（LaunchAgent，35001）、mazu/athena/cthulhu/valkyrie（NFS 共用 binary，各自 systemd，35001）、Oracle（systemd，35001）、Zeus（`swear02` systemd，35002）。swop 因離線未安裝。
- **Zeus 注意事項**：Zeus 正式運行由 `swear02` 擁有，監聽 `35002`，home 目錄權限 `700`；更新時須從 `<remote-home>` 跨帳號暫存或透過 SSH alias `su_zeus` 操作。
- **SHA-256**：
  - Mac arm64：`725ab7956b7aadc873aac54d7206537431d736a05254b83a3e27815960ac5cc1`
  - Linux x64：`a99874c9cc6ff47b00c4c18fd6efeb1c49a47d341e91653e7c72373cd57e79f8`
  - Linux ARM64：`1a02f1eee445c08fc2a419d55404cfc4464a6a56bc82f38b9e1b3852976eeba3`
  - Windows x64：`09132180fb53a08999ef22a0ae282155c0080eda0847faca2c2711f7f3731a37`（已建置未安裝）
- **真實驗證**：七台主機以 `User-Agent: Python-urllib/3.14` 送出真實 `deepseek-flash` 推論，皆成功穿透 Cloudflare，HTTP 200 回應於 `opencode-go-1`（`X-Gateway-Attempt: 1`）。

