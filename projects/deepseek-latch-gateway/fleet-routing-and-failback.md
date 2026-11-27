---
title: DeepSeek Latch Gateway fleet routing, failover, and rollout pitfalls
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: Bun/systemd/launchd/Windows-Task-Scheduler
status: active
created: 2026-08-25
updated: 2026-11-28
tags: [gateway, opencode-go, routing, failback, hapi, swear-review, model-alias, openrouter]
---

## 呼叫正確的 gateway，驗證真正的 upstream

- 每個 client 使用同機 loopback gateway。AIsimpV／Pi 的 DeepSeek 路徑應保留 local gateway，即使環境提供 `DEEPSEEK_API_KEY`。Pi 的 `opencode-go` provider 通常是 `http://127.0.0.1:35001/v1`，用公開 placeholder `local-gateway`。官方 API 的 402／餘額屬於不同帳戶。
- 2026-10-25 的簡化架構使用一份 `config.yaml`，其中 `providers` 定義連線與 compat，`models` 為 public id 對應有序的 `{provider, model}` deployments。新舊 binary 與 config 格式互斥，遷移須一起換、一起回復。舊 `routing.yaml`、priority group、latch、half-open probe、指數 cooldown、`/switch` 已退役。看到舊 config、指令或 UI 時先核對 live binary 和有效 config。
- 設計上 `deepseek-flash` 依序 OpenCode Go 槽位、Command Code、OpenRouter latest，`deepseek-pro` 依序 Command Code、OpenRouter latest。舊的 DeepSeek 版本／日期／前綴 ID 在相容層映射到這兩個 public id，無法用來鎖 snapshot。`-fast`、`-vision-exp` 或無關名稱不在相容範圍。
- 記錄 `X-Gateway-Active-Endpoint`、`X-Gateway-Attempt`、`X-Gateway-Upstream-Model` 和 non-streaming response 的 `model`、provider、usage cost。HTTP 200 可能來自 fallback，長批次可在排程中換模型並新增費用。HTTP errors 也留 routing metadata。Gateway 可改寫 model、strip `response_format`、正規化 reasoning，client 原始 JSON 無法證明 upstream 實收內容。
- 目前需求是 DeepSeek 最新模型。用 `bun run check-latest <config>`（載入 service 使用的 secret 來源）比較 provider 目錄，再實際呼叫 OpenRouter latest alias。OpenRouter 用 `canonical_slug`／`created` 與 served model 驗證，undated ID 和日期字串都不足以證明 snapshot。Command Code／OpenCode Go 只有 undated 名稱時，最多能驗證當時的 catalog 版本。用已知 modality fixture 輔助辨認，不採信模型自述。價格上限按 provider 實際費率評估，過低會排除所有候選。
- 改模型 ID 時保留其他 provider 與模型。Pi 要對齊 `models.json`、`settings.json` 的 `enabledModels`、`model-filter.json` 及 `strict-model-allowlist.ts`（含 `includes` 特例）。Picker 可能受 local override 影響。用 `session_start` 的 `ctx.modelRegistry.getAll()` 核對完整集合，再實際推論。OpenCode CLI 也有 `model`、`small_model`、provider whitelist，dsh 有自己的 advisory catalog。備份名稱即使寫著 pre-rename，仍須比對 mtime 與內容。見 [Pi 模型改名](../../tools/pi/deepseek-model-id-migration.md)。

## Failover、quota 和 session header

- Provider failure 對 **provider + model** 設 30 分鐘 cooldown。分類包含 429／402／quota 字樣、401、OpenCode 訂閱 403、Cloudflare 1010、408、5xx，網路錯誤須連續兩次。一般 4xx 透傳，client 斷線不 cooldown。到期由下一個 request 重試。全部在 cooldown 時回 503 `no_available_provider` 加 `Retry-After`，不提前探測。看 `/status` 各 deployment 的 failures、`cooldown_until`、`last_error`。舊版 `circuitState: open` 曾在 `blockedUntil` 已過期時仍顯示 open。
- Request 帶穩定的 `x-opencode-session`。缺 ID 時以 client identity 與開場訊息產生 affinity，同一開場可能共用，compaction 改開場也會改 affinity。兩者都缺時，現行 gateway 會跳過 OpenCode deployment 且不做 cooldown。看到 upstream 400 `MissingSessionID` 時先查實際送出的 header。
- OpenCode Cloudflare 403／1010 與一般權限 403 分別處理。隔離測試證實舊 binary 遇 1010 直接返回、429 才切來源。更新加入 generic client UA 正規化（如 `Python-urllib`）和 1010 專用 failover。重裝相同舊 binary 不能補 classifier；隔離 fixture 可證實封包／切換，真實上游成功另用帶 session ID 的請求驗證。
- OpenCode Zen `CreditsError`／`Insufficient balance` 可用 401 回傳，與 Go 週期 quota、訂閱 workspace、Command Code 額度分開確認。OpenCode CLI 與 gateway slot 若載同一 credential 就共用額度。401 `api_key_expired` 遇 provider 全面撤銷時需換 key，重新登入不會復原舊 key。Endpoint 編號是槽位，不能按歷史帳號名稱推測當前 key。

## Fleet 變更與真實驗證

1. 核對 live gateway instance、監聽 port、supervisor、binary 路徑、config 及 secret 來源。Unix NFS 共用檔案可能從不同 mount 看見，仍是同一份；只修改一次、逐 process 驗證。Zeus 曾同時有 `zeus`／swear01 的 35001 與 `su_zeus`／swear02 的 35002，兩個 home 要分別核對。Mac 是 arm64 LaunchAgent，Oracle 是獨立 arm64 systemd user service。SWOP 即使不在 HAPI machine 清單仍須納入，核對 SYSTEM 的 `DeepSeek Gateway (SWOP)` task 與 `<program-data>/DeepSeekGateway/deepseek-gateway.exe`，也同步使用者 `.local/bin` 副本。
2. 用同目錄 staging、SHA 校驗和原子替換，保留已知可回復備份。舊 binary 可能根本不 parse `extra_body`，僅改 YAML 無法修復。確認 config endpoint 與程序環境都到位。不要從固定 package version、Swear Review caller 的部署或磁碟有新檔推斷 live gateway 已更新。用 Linux `/proc/<pid>/exe` 比對 SHA；Mac／Windows 核對新程序路徑、啟動時間及安裝後 hash。
3. 等 established gateway 連線結束，只重啟 gateway supervisor，保留 HAPI Runner 和現有 session。NFS 同一份 log 可能混多機輸出，對照新 PID 和 stdout offset。隔離 smoke port 要等實際 bind，connection refused 的無等待迴圈會太早耗盡。
4. 驗證 `/status` 後，使用實際 model 和穩定 session header 推論，同時檢查 status 與 active endpoint。輪替 Command Code 以 `deepseek-pro` 驗證並要求 active endpoint 為 `command-code`。要證實 OpenRouter fallback，可在暫存隔離 config 指向失敗的 loopback upstream 並保留 OpenRouter，使用該進程的 PID 停止 smoke。避免在 shell 指令本身含暫存路徑時用同一 pattern `pkill -f`。只回報已逐機驗證的覆蓋範圍。
5. 換 key 檢查 service runtime，更新檔案不保證程序已重載。以前的拼字 `COMMAMD_CODE_API_KEY` 要從 config、啟動腳本和 Windows DPAPI 名稱一併遷移到 `COMMAND_CODE_API_KEY`。Oracle PM2 process env 須用更新後的環境 `pm2 restart <name> --update-env`。OpenCode 在既有工作流可能同時用 `.secrets` 與 `.dsh/.credentials.yaml`。用變數引用保存 config，key 值留在核准的憑證來源。隔離測試未載 secret 時，401 `Missing Authentication header` 首先檢查 smoke 進程環境。

完整發版、帳戶槽位快照、舊 router 行為、測量值和逐機驗證收據，見 [整理前固定版本](https://github.com/swear01/agent-memory/blob/d1e08b2dad393fd2ecf9dbbee4f69d53bd6d8755/projects/deepseek-latch-gateway/fleet-routing-and-failback.md)。
