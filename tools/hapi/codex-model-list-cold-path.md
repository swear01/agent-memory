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

## 分別量 Runner 與 Codex 的兩層快取

- `Codex app-server request 'initialize' timed out after 30000ms` 對應當時 HAPI initialize 時限。先分別量 spawn、initialize 與 `model/list`，冷查詢慢而熱查快只能定位到整條冷路徑。早期版本的 5 分鐘快取是 Runner process 內、下一次 demand 時更新，沒有每五分鐘啟動一次的 timer。
- Codex CLI 0.157.1 實測：同一 app-server 改 `model_catalog_json` 後再發 `model/list` 仍見啟動時舊 catalog。Runner 結果 TTL 到期不能強迫另一層長駐程序重載。與 [模型 allowlist 和版本查核](codex-model-catalog-allowlist.md) 一起讀。
- 歷史 [HAPI #1938](https://github.com/tiann/hapi/pull/1938) 候選：machine RPC 按需求啟動，只快取成功的非空結果；以 `CODEX_HOME/config.toml`、`auth.json` 和 catalog 指紋失效，相對 catalog 依 config 位置定位。該候選 TTL 24 小時；沒有本機檔變更的 provider 變動可能延到 TTL 才可見。Shared session 可優先用自身 `model/list`；session RPC 缺失用 `503 rpc_target_missing` 讓 Web 回退 machine，通用 500 不可一律回退。Create 頁 Default 不能等清單阻塞。以所用 release 確認有無候選修正，不能沿用歷史 PR CI。
- 歸因時查真實 endpoint、進程與 catalog 讀回；不拿 `models_cache.json` 代替實際 catalog，也不預設定時重啟就是修正。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/codex-model-list-cold-path.md)
