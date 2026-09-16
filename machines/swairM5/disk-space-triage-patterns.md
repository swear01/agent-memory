---
title: "SwairM5 macOS 磁碟容量排查與主要佔用模式"
scope: "machines/swairM5"
status: active
updated: 2026-09-16
---

# SwairM5 macOS 磁碟容量排查與主要佔用模式

## Problem

在 `SwairM5`（1TB APFS SSD）上，可用空間耗盡至極低水準（例如小於 30 GB）時，空間往往集中於隱蔽的開發暫存、媒體修復與工作樹目錄，常規清理工具難以直接辨識。

## Durable rule

排查磁碟空間時，應優先針對以下 4 大元兇進行掃描：
1. **Codex 批次作業暫存**：`<documents-root>/Codex/<date>/.../work` 下可能存在數百 GB 的中繼處理檔案（如 Takeout 批次修復、圖片暫存）。完成任務後應優先回收。
2. **跨主機除錯與臨時 Clone**：`/private/var/tmp` 會長期殘留歷史多機測試、QMD 或 issue 除錯複本（如 `qmd-issue25-*`、`shared-memory-*`），重開機亦不會自動清除，單次可釋放數十 GB。
3. **Agent 工作樹與殘留構建包**：`<agent-worktrees-root>` 容易堆積大量已合併或過期分支，且常包含 loose log/tar.gz 檔案與 Unity/C# 編譯快取。
4. **套件與下載快取**：定期執行 `brew cleanup -s` 清空 Homebrew Cask 與 Bottle 下載暫存，並清理 `<remote-home>/Downloads` 的重複壓縮檔。
