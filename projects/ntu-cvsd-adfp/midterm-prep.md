---
title: CVSD 期中考古題準備（2023 fall 題目重打版）
scope: projects/ntu-cvsd-adfp
status: verified-local; cloud-sync-unverified
updated: 2026-11-03
---

# CVSD 期中考古題準備

## 成果位置（2026-11-03 驗證）

- 學習用副本：`<GoogleDrive-root>/document/學習紀錄/碩一/電腦輔助積體電路設計/midterm-prep/CVSD_2023_fall_midterm_questions.pdf`（15 頁，只有題目，無手寫答案或批改）。該資料夾 README.md 已加一行索引。
- 同檔另放講義區：`<GoogleDrive-root>/document/學校講義/碩一/電腦輔助積體電路系統設計/考古/CVSD_2023_fall_題目.pdf`（與 `CVSD_2015.pdf` 並列）。
- `<GoogleDrive-root>` = 使用者指定帳號的 DriveFS「我的雲端硬碟」。兩份用 `cmp` 核對本機逐位元組相同；Google Drive 雲端同步未驗證。

## 來源與已知缺漏

- 來源是 19 張手機照 `page-01..19.png`，位於 Codex 工作區（thread 01a199d8-bb16-7e70-8ae4-6a4a3f0eb461，2026-11-02）；該暫存區不是正式保存位置。
- 照片缺封面與 Problem 5（原卷第 13–15 頁）；SDF TIMINGCHECK 最後兩行被截掉，重打版以 `...` 標示。
- Problem 1、Problem 3 的波形轉折時間因照片歪斜，是從程式碼推回來的；area report 的 buf/inv 數讀成 60（也可能是 68）。引用這些數字前要對照原照片。

## 方法教訓

- 有手寫痕跡的考卷照片，用 AI 圖像擦除（imagegen）會破壞印刷的程式碼，已放棄。
- 可行做法：把文字重打成 HTML，波形與電路用 inline SVG 重畫，再用 headless Chromium 列印成 PDF：`/Applications/Chromium.app/Contents/MacOS/Chromium --headless=new --no-pdf-header-footer --print-to-pdf=<out.pdf> <in.html>`。
- 產生器腳本在臨時 scratchpad，沒有保留；要改版需重做。

## 使用者進度

- 使用者已開始練習 Problem 1 Q1（Verilog timing control：fork/join、wait、negedge counter），尚未檢查答案。
