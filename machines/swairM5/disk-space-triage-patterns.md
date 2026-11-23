---
title: "SwairM5 macOS 磁碟容量排查與主要佔用模式"
scope: "machines/swairM5"
status: active
updated: 2026-11-23
---

# SwairM5 macOS 磁碟容量排查與主要佔用模式

## Problem

在 `SwairM5`（1TB APFS SSD）上，可用空間耗盡至極低水準（例如小於 30 GB）時，空間往往集中於隱蔽的開發暫存、媒體修復與工作樹目錄，常規清理工具難以直接辨識。

## Durable rule

排查磁碟空間時，應優先針對以下 8 大元兇進行掃描：
1. **Codex 批次作業暫存**：`<documents-root>/Codex/<date>/.../work` 下可能存在數百 GB 的中繼處理檔案（如 Takeout 批次修復、圖片暫存）。完成任務後應優先回收。
2. **跨主機除錯與臨時 Clone**：`/private/var/tmp` 會長期殘留歷史多機測試、QMD 或 issue 除錯複本（如 `qmd-issue25-*`、`shared-memory-*`），重開機亦不會自動清除，單次可釋放數十 GB。
3. **Cursor 專案快照**：`<remote-home>/Library/Application Support/Cursor/snapshots` 會隨 Agent/Composer 多專案操作累積 10~20+ GB 的 shadow codebase，可安全清空。
4. **Agent 工作樹與殘留構建包**：`<agent-worktrees-root>` 容易堆積大量已合併或過期分支，且常包含 loose log/tar.gz 檔案與 Unity/C# 編譯快取。
5. **Remotion / 多媒體專案輸出**：多媒體專案的 `out/`、`build/` 及 `public/clips/` 常累積數 GB 渲染影片，搬移至雲端硬碟前應先清理並移除 `node_modules`，以防同步卡死。
6. **套件與下載快取**：定期執行 `brew cleanup -s` 清空 Homebrew Cask 與 Bottle 下載暫存，並清理 `<remote-home>/Downloads` 的重複壓縮檔。
7. **Unity 編輯器**：Hub 版本在 `/Applications/Unity/Hub/Editor/<version>`，單一版本約 19 GB（編輯器 7.8 GB、PlaybackEngines 12 GB，其中 WebGL 6.3 GB）。`/Applications/Unity/Unity.app` 是 Hub 以外的舊獨立安裝（曾是 2022.3，7.6 GB），清舊版時要一併檢查；它旁邊的 `PlaybackEngines` 屬 root，需使用者自行 `sudo rm`。
8. **PrismLauncher**：instance 的 `minecraft/magic_storage_backups`、`auto_storage_backups` 會累積成數 GB 的模組自動備份。

## 2026-11-23 清理紀錄

- `<documents-root>/Codex` 曾佔 199 GB。主因是 Drive 影片壓縮任務在 `work/` 留下 `original.*`、`candidate.*`、`cloud-readback.*`（`result.json` 的 `phase: complete_replaced` 表示雲端已換成壓縮版）。刪除前要告知使用者：原畫質沒有其他備份，B2 也只存壓縮版。
- 一次清理後可用空間從 288 GB 升到約 500 GB。剩下的約 393 GB 沒有單一大項，分散在 `~/Library`（112 GB）、Documents 專案、`/Applications`（55 GB）、開發工具快取（約 60 GB）、Homebrew（21 GB）。
- 刪 worktree 或 repo 前先跑 `git status` 和 `git log --branches --not --remotes`；有未推送 commit 就保留。
