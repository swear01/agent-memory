---
title: "Agent 工作樹稽核應處理巢狀結構與孤立父儲存庫"
scope: "tools/git"
status: active
updated: 2026-09-16
---

# Agent 工作樹稽核應處理巢狀結構與孤立父儲存庫

## Problem

在 `<agent-worktrees-root>` 下進行工作樹清查時，常見以下誤判與殘留問題：
1. **巢狀工作樹結構**：部分 AI Agent 或多平台專案會將工作樹建立於一階子目錄中（如 `<task-name>/<platform-name>/.git`）。若稽核腳本僅檢查頂層目錄是否存在 `.git`，會將此類工作樹誤判為 `not_a_worktree`，可能導致誤刪或統計失真。
2. **孤立工作樹（Orphaned Worktree）**：當位於 `<documents-root>/Projects/<repo>` 的主儲存庫被刪除時，工作樹中的 `.git` 檔案仍指向原路徑，導致 Git 命令拋出 `fatal: not a git repository`。此時無法透過主儲存庫執行 `git worktree list` 或 `git worktree remove`。
3. **引擎與編譯快取膨脹**：Unity（`Library/`、`Builds/`）或 .NET（`bin/`、`obj/`）等專案，工作樹體積常達 3~9 GB，但真正的未提交代碼修改往往僅數百 KB。在遵守「保留 dirty 工作樹」原則時，若任由巨型快取常駐，會嚴重浪費磁碟空間。

## Durable rule

1. **多層級 Git 檢查**：稽核工作樹時，若頂層無 `.git`，需額外檢查一階子目錄。
2. **孤立工作樹判定**：讀取 `.git` 內的 `gitdir:` 指向，確認主儲存庫實體是否存在；若主儲存庫已不存在，應標記為 orphaned 並在確認無未保存資料後直接安全清理。
3. **Dirty 龐大工作樹保全與瘦身**：當 dirty 工作樹主要空間被產物（如 Unity Library）佔據時，先將未暫存修改 commit 至分支或匯出 patch，隨後即可安全移除整個龐大工作樹，兼顧代碼不遺失與空間釋放。
