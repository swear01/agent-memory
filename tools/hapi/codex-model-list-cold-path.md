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

HAPI `cli/src/modules/common/codexModels.ts` 對非空 `model/list` 成功結果做每個 Runner 進程各自的 5 分鐘快取與併發合併。快取失效時，會建立新的 `codex app-server`、執行 `initialize` 和 `model/list`，然後關閉子進程。Web `useCodexModels` 有 machine ID 時優先查 machine RPC，即使 shared Codex session 已有可提供 `model/list` 的連線。新建 session 只能走 machine RPC。

2026-09-28 經 HAPI machine API 量測：cthulhu 冷查詢約 14.8 秒，立即重查低於 0.2 秒；這只能證明整條查詢路徑冷慢，不能區分 spawn、initialize、model/list 的耗時。`Codex app-server request 'initialize' timed out after 30000ms` 的 30 秒來自 HAPI `CodexAppServerClient.initialize()`。

# 長駐連線的正確性限制

這個 fleet 以 `model_catalog_json` 指向本機 allowlist 管理可見模型。2026-09-28 用已安裝的 Codex CLI 0.157.1 與隔離的臨時 `CODEX_HOME` 實測：同一個 app-server 先讀只含 `gpt-daybreak-blue-latest` 的 catalog，將該檔案改為只含 `gpt-6-astra`，再次呼叫 `model/list` 仍回傳舊模型。這與 openai/codex#35129 的未結案回報相符。因此不能只把 HAPI 的 machine 或 shared session 模型查詢改為長駐連線，否則 5 分鐘的 HAPI 結果快取過期也無法保證讀到新 allowlist。官方 app-server daemon 同樣長駐，且其生命周期契約仍屬實驗性。

2026-09-28 進一步比較：`agentclientprotocol/codex-acp` 在一個 ACP 進程持有一條 Codex 連線，建立或恢復 session 時直接用該連線呼叫 `model/list`；OpenClaw 預設使用可借用的 shared app-server client 查模型；Plannotator 的模型探測用短命程序，成功後的清單保留在服務進程中，不按五分鐘重新啟動。這些模式都沒有要求每五分鐘定時 initialize。HAPI 目前也沒有定時器，僅在快取過期後的下一次查詢重新啟動。

若以避免重複啟動為首要目標，先考慮延長或取消 HAPI 的固定 TTL，改由已知輸入變更（Codex 可執行檔、config、catalog、認證）與明確的手動刷新失效；遠端模型變更的自動新鮮度須另外決定。若改為長駐連線，須在 catalog/config 更新時重啟 app-server，且處理閒置關閉與程序異常；否則本機實測的舊清單問題仍在。先記錄 spawn/initialize/model/list 各段耗時。不要改讀 `models_cache.json` 或無條件設計五分鐘背景重啟。

參考：OpenAI Codex app-server 文件的 connection lifecycle / `model/list`、官方 Python SDK `AsyncCodex` 連線生命周期、官方 `codex-app-server-daemon` README、openai/codex#35129。
