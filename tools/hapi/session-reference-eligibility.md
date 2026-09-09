---
title: HAPI session 引用資格應依對話內容判斷
scope: tools/hapi
status: verified
updated: 2026-09-09
---

## 根因與需求

- tiann/hapi #1506 / #1507 為排除啟動失敗空殼，將 `hasSessionTitleSignal`（metadata.name 或 metadata.summary.text）當成引用資格。#1418 拖曳分支也沿用此條件。
- #1799 修正需求：未命名但有對話仍可引用；命名空殼仍排除。顯示名稱、path / ID fallback、搜尋排序與資格須分開。
- `output.data.type = summary` 是標題事件（`apiSession.ts` 會據此寫 metadata.summary），不能視為對話內容。不要與 compact-summary 混淆。

## 已驗證資料流

- 訊息內容可能是 zstd BLOB；不可用 SQL JSON 函式或原始 message count 代替實際內容判斷。role=user 文字／附件與 agent 對話／工具內容可計入；啟動、切換、usage 等 bookkeeping 不可計入。
- SessionCache.refreshSession 覆蓋 reload / 多數 import、merge、rewind，但一般新訊息不一定經過 refresh。Web／Telegram send 與取消會直接經 EventPublisher，不能只改 SyncEngine.handleRealtimeEvent。
- Codex direct import 先建立 cache，再追加內容；新建 session 不發逐筆 message-received，需要寫入完成後 refresh。
- 發布內容資格變更的完整 session 時，使用當下物件快照，避免後續 markMessageQueued 修改同一物件，令 earlier event 的 thinking 狀態被回溯改寫。

## SQLite iterator 注意

本機 Bun 1.3.14 實測：使用快取的 db.query(...).iterate()，找到第一筆即 return，重複查詢可能報 `bad parameter or other API misuse`。改用獨立 db.prepare，並以 try/finally 呼叫 statement.finalize；仍逐筆 decode 並於首筆有效內容停止，不載入完整 transcript。
