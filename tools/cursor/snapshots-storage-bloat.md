---
title: "Cursor snapshots 專案快照膨脹與安全清理"
scope: "tools/cursor"
status: active
updated: 2026-09-16
---

# Cursor snapshots 專案快照膨脹與安全清理

## Problem

在 Cursor 中使用 Agent 或 Composer 編輯大型專案時，Cursor 會在 `<remote-home>/Library/Application Support/Cursor/snapshots` 目錄下自動建立專案代碼樹檢查點：
- `snapshots/codebases/`：存放專案完整樹狀結構，單一大型專案多次修改會產生多份副本（如大型儲存庫 4 次快照即佔用 11 GB 以上）。
- `snapshots/stores/`：存放內容雜湊（Content-Addressable Blobs）。
隨著使用時間增長，`snapshots` 單一目錄常累積 15~30 GB 以上，嚴重佔用本機磁碟。

## Durable rule

1. **快照定位與性質**：`snapshots/` 僅用於 Composer 歷史還原與即時比對，不包含使用者設定、擴充功能、歷史聊天記錄或驗證金鑰。
2. **清理安全性**：若無復原過去 Composer 編輯點的需求，清空 `snapshots/*` 完全安全，不會影響 Cursor 的正常啟動、擴充功能運作與個人設定。
3. **排查時機**：在 macOS 遭遇不明磁碟空間吃緊時，應將 Cursor `snapshots` 列為重點檢查目標。
