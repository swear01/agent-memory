---
title: Codex Luna Reserve — HAPI 控制邊界與條件式 Usage 顯示
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
updated: 2026-11-28
---

## 額度與 UI

- **Usage 有 Reserve bucket，不代表正在使用 Reserve。** 一般使用時隱藏；目前實際進入 Reserve 才顯示，Reserve 耗盡時保留耗盡狀態，離開後隱藏。普通 Luna、Reserve 和 Fast／Standard 是各自的值。Reserve 不設為全域模型預設。
- **一般額度百分比見底，仍需等 native 資格。** `ordinaryUsageAllowed=null` 代表未知，不能由百分比或 reset 時間推算 entitlement。讀後端允許與 Reserve banner。按 limit ID 合併快照；Reserve 曾在 `base_model_inference` bucket，應核對 `limitName=gpt-reserve`，不可覆蓋普通 `codex` 週額度。缺值顯示未知。
- **RPC ACK 後 picker 又回舊值。** HAPI 遠端 app-server 由 session 設定送每輪；改動需等 `thread/settings/updated` 確認並與 resume／handoff 同步。歷史自動切換 PR #1780 已撤回，#1836 是條件式手動切換；native TUI 的 fallback 則有自己的規則。不可依 picker 發出動作重播失敗回合。

## 驗證

- `account/rateLimits/read` 先用 null 探測。0.153.4 對 `supportsLunaReserve:false` 回 -32600；0.154.0 有新版欄位。只有完整實作 client 端接續時才宣告 supports。用真實 `thread/settings/updated` 測延遲確認，RPC ACK 與 native 事件分開。
- [#1836](https://github.com/tiann/hapi/pull/1836) 當時的實測：`model/list(includeHidden=true)` 沒列 `gpt-reserve`，但隔離 thread 的原生 Reserve 推論成功且回獨立 bucket，百分比未變。這不能證明 quota 增量，也不能證明 HAPI 條件式切換已端到端部署；應用真實符合資格的 HAPI session 核對 picker、native model 與扣額，不刻意耗盡額度。

[整理前完整證據](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/tools/hapi/codex-luna-reserve.md)
