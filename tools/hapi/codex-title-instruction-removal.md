---
title: HAPI Session 命名：原生名稱、觸發與 Hub 同步
scope: tools/hapi
status: active
updated: 2026-11-28
---

名稱問題先分清終端標題、原生 session name、第一則訊息的 fallback、Hub metadata 與手動名稱。沿 picker／sidebar 的來源、adapter、producer 和觸發檢查。工具已註冊或 adapter 有 receiver，只證明該步有程式碼；要讀 native state、raw event 和 Hub 可見值核對同一個 session。手動名稱仍優先。

## Codex remote 的歷史案例

- HAPI v0.29.0.5 曾注入 HAPI title 指令；v0.29.0.6 已移除，但仍保留 `change_title` bridge。Zeus 當時的 Codex 0.153.0 在實際 `sqlite_home` 讀到 `threads.name=NULL`，`title` 是第一個 prompt，並非單純 HAPI 顯示漏讀。先比對 deployed tag、rollout 與該 host 的 DB 路徑。
- Codex 0.153.0 的 TUI 以 temporary thread 生成名稱再 set；headless app-server 不會因啟動 TUI 的原生功能就自動套用。Hub 的手動 titleSuggestion 也不同。歷史 #1771 在隔離 runtime 驗證 remote 接 `thread/name/updated`／start-resume-fork 回傳名，local 輪詢 `thread/read`。remote 在第一個完成工作後仍無名，用 ephemeral、read-only、停用工具/MCP/plugins/hooks 的生成 call 再 `thread/name/set`，不再往主對話注入 title prompt。這是當時分支測試，未當成正式部署或現行協定保證。
- thread 尚未持久化時跨 app-server `readThread` 可回 `thread not loaded`。先用 start/resume 的結果，跨程式查證在主 thread 有實際一輪後做。用相同 thread ID 比對 `thread/read.name` 與 metadata summary；生成總 deadline 包含 initialize/config/model/read/start/turn/set，最後關掉臨時 client 與停止不再使用的 poller。

## Pi 名稱沒有同步

Pi 的 session name、terminal title 與模型摘要各自驗證。曾有命名套件命令已註冊但 name=None，沒有證明真實觸發或同步。核對 extension 載入、實際命名觸發、session name 是否產生，再核對 Adapter 是否消費該 RPC 事件。擁有 fallback 顯示名無法證明原生摘要已完成；當時原因和修復未經 runtime 證實。

[Codex 原始案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/codex-title-instruction-removal.md) · [Pi 名稱來源](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/pi-title-identity-layers.md) · [Pi 觸發案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/native-name-generation-boundary.md)
