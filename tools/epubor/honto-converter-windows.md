---
title: Honto 電子書除 DRM 轉檔（Windows + Epubor Honto Converter）
scope: tool-specific
tool: Epubor Honto Converter
status: active
created: 2026-09-21
updated: 2026-09-21
tags: [honto, epubor, drm, windows, ebook, japanese]
---

# Honto 電子書 → PDF（Windows）

驗證過的完整流程：Honto Windows App 下載書籍後，用 **Epubor Honto Converter**（NSIS 安裝，v1.3.x）除 DRM 並輸出 PDF。

## 關鍵事實（都踩過坑驗證）

- **下載連結在日文站**：`https://jp.epubor.com/honto-converter/`（下載：`https://jp.epubor.com/download/epubor-honto.exe`）。舊的 `www.epubor.com/honto-ebook-converter/` 已 404，不要再找舊站。
- 安裝：NSIS，需管理員權限。靜默：`/VERYSILENT /NORESTART /allusers`（要 UAC，用 PowerShell `Start-Process -Verb RunAs -Wait`）。裝到 `C:\Program Files (x86)\Epubor\EpuborHontoConverter\`。
- **Honto App 的書下載位置**：`<userprofile>\AppData\Local\DNP\honto\Contents\<user-id>\*_<DIVF4|EPUBX>\`。`_DIVF4`（固定版面）和 `_EPUBX` 都支援；`_BOOK` / `DIVF2` / `EPUB2IV` 不支援，要從本棚刪掉重新下載。
- Converter 自動偵測 Honto App 下載的書（日誌 `getBooks` / `allItems count`）。程式是 GUI + 內含 Python，**沒有 command-line 轉換模式**，最後的同步/轉換要用 GUI 點。
- 轉檔操作：選中書列（左欄單擊一次即可，不用拖曳）→ 右側出現輸出選項 → 轉換。
- **輸出結構**：`<輸出目錄>\<書名>\` 內含 `<書名>.pdf`（每頁一張圖打包成 PDF）+ `images\001.jpg...N.jpg`（每頁原圖）。固定版面教材轉出來是**圖片式 PDF**，不能全文搜字，但可備份/傳輸/列印。
- 首次開啟可能先彈註冊/登入對話框；試用版免費額度 3 本。

## 可複製的自動化腳式（GUI 操控）

- 截圖：PowerShell `System.Drawing` `CopyFromScreen`（`PrintWindow` 對此 app 抓到黑圖，不要用）。
- 視窗座標：`GetWindowRect` + `ShowWindow(3)` 最大化後座標固定，再截圖裁剪定位按鈕。
- 點擊：AutoHotkey v2（`C:\Program Files\AutoHotkey\v2\AutoHotkey.exe <script.ahk>`），`MouseMove x,y; Click`。
- Muse（meta_analyze_file）對「DRM 除鎖」字眼會拒絕給座標；改問「OCR + 純位置」會正常回座標。

## 實例

- 2026-09-21：《大学の日本語 初級 ともだち Vol.2［第2版］》（Honto ¥2,750，DIVF4，133MB）→ 370 頁 PDF（79.6MB），放 `G:\My Drive\document\學校講義\碩一\日文二上\大學的日本語 ともだちVol.2 課本.pdf`。