---
title: HAPI Codex 模型列表冷路徑與 catalog 快照
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
created: 2026-09-28
updated: 2026-09-28
tags:
  - hapi
  - codex
  - model-catalog
  - startup
---

# 已驗證的路徑

修正前 HAPI `cli/src/modules/common/codexModels.ts` 對非空 `model/list` 成功結果做每個 Runner 進程各自的 5 分鐘快取與併發合併。快取失效時，會建立新的 `codex app-server`、執行 `initialize` 和 `model/list`，然後關閉子進程。Web `useCodexModels` 有 machine ID 時優先查 machine RPC，即使 shared Codex session 已有可提供 `model/list` 的連線。新建 session 只能走 machine RPC。

2026-09-28 經 HAPI machine API 量測：cthulhu 冷查詢約 14.8 秒，立即重查低於 0.2 秒；這只能證明整條查詢路徑冷慢，不能區分 spawn、initialize、model/list 的耗時。`Codex app-server request 'initialize' timed out after 30000ms` 的 30 秒來自 HAPI `CodexAppServerClient.initialize()`。

# 長駐連線的正確性限制

這個 fleet 以 `model_catalog_json` 指向本機 allowlist 管理可見模型。2026-09-28 用已安裝的 Codex CLI 0.157.1 與隔離的臨時 `CODEX_HOME` 實測：同一個 app-server 先讀只含 `gpt-daybreak-blue-latest` 的 catalog，將該檔案改為只含 `gpt-6-astra`，再次呼叫 `model/list` 仍回傳舊模型。這與 openai/codex#35129 的未結案回報相符。因此不能只把 HAPI 的 machine 或 shared session 模型查詢改為長駐連線，否則 5 分鐘的 HAPI 結果快取過期也無法保證讀到新 allowlist。官方 app-server daemon 同樣長駐，且其生命周期契約仍屬實驗性。

2026-09-28 進一步比較：`agentclientprotocol/codex-acp` 在一個 ACP 進程持有一條 Codex 連線，建立或恢復 session 時直接用該連線呼叫 `model/list`；OpenClaw 預設使用可借用的 shared app-server client 查模型；Plannotator 的模型探測用短命程序，成功後的清單保留在服務進程中，不按五分鐘重新啟動。這些模式都沒有要求每五分鐘定時 initialize。HAPI 目前也沒有定時器，僅在快取過期後的下一次查詢重新啟動。

2026-09-28 已提交上游 issue tiann/hapi#1937 和 PR #1938（head `3162a904d`，CI 與自動 review 通過；`swear01` 無 upstream merge 權限，故當時尚未合併）。修正方案：machine 查詢只在有需求時啟動 app-server；成功且非空的模型清單在 Runner 進程內快取 24 小時；每次查詢前對 `CODEX_HOME` 的 `config.toml`、`auth.json` 與 `model_catalog_json` 指向的檔案做指紋，變更即失效。相對 catalog 路徑依 `config.toml` 所在目錄解析。共用 Codex session 優先使用自身已有的 app-server `model/list` RPC；舊式 session 仍走 machine，session RPC 目標不存在時 hub 回 503 `rpc_target_missing` 讓 Web 回退 machine。沒有背景刷新或手動刷新按鈕；新建 session 使用 Default 模型不必等清單。遠端模型變動在未改變本機檔案時最多可延遲 24 小時顯示，Runner 重啟後首次 machine 查詢仍可能冷慢。

此修正曾被自動 review 指出兩個真實缺口：相對 catalog 路徑未列入指紋，以及 session RPC 缺失時 hub 原本回無 code 的 500。修正後最新 head review 無可執行缺陷，完整 CI 的 integration、test、windows-codex-mcp、pr-review 全部通過。若未來要進一步消除 Runner 首次冷啟動，先量測 spawn、initialize、model/list 各段耗時；不要改讀 `models_cache.json` 或無條件設計五分鐘背景重啟。

參考：OpenAI Codex app-server 文件的 connection lifecycle / `model/list`、官方 Python SDK `AsyncCodex` 連線生命周期、官方 `codex-app-server-daemon` README、openai/codex#35129。
