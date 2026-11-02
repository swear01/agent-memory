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
- 原照片上有學生手寫作答與紅筆批改（Problem 2 的註記、Problem 7 setup/hold 表打勾），可當解答參考；Problem 7 的推算與批改答案一致。
- 2026-11-03 修正：Problem 2 重打版原本結尾多一個 `end`，原卷是三個 `end` 收 always、缺 `endmodule`；兩份 PDF 已重產並覆蓋。
- Problem 1、Problem 3 的波形轉折時間因照片歪斜，是從程式碼推回來的；area report 的 buf/inv 數讀成 60（也可能是 68）。引用這些數字前要對照原照片。

## 方法教訓

- 有手寫痕跡的考卷照片，用 AI 圖像擦除（imagegen）會破壞印刷的程式碼，已放棄。
- 可行做法：把文字重打成 HTML，波形與電路用 inline SVG 重畫，再用 headless Chromium 列印成 PDF：`/Applications/Chromium.app/Contents/MacOS/Chromium --headless=new --no-pdf-header-footer --print-to-pdf=<out.pdf> <in.html>`。
- 產生器腳本在臨時 scratchpad，沒有保留；要改版需重做。

## 解題慣例（以原卷批改為準）

- STA：輸入/輸出延遲相對理想 CLK；Tclkx 是 clock latency，兩顆暫存器相同。暫存器端 setup required = T + Tclkx − Tsu，hold required = Tclkx + Th；輸出端 setup required = T − DO，hold required = −DO；暫存器起點 arrival 從 Tclkx + Tco 起算。
- SDF：IOPATH 兩組括號是 output 的 rise/fall（不是 input），每組是 (min:typ:max)；max delay 取第三個，fast corner 取第一個。

## 使用者進度（2026-11-03）

- 已和使用者逐題走完所有有照片的題目（Problem 1–4、6、7）。
- 弱點：fork 與 begin…end 的 `#` 延遲累加差異、運算式寬度與符號擴展（未標寬度常數是 32-bit signed）、SDF rise/fall 看 output、低溫在不考慮 temperature inversion 時變快、DesignWare 架構選擇是 area/speed trade-off 的關鍵字、WLM 估 interconnect area。
- 待補：Problem 5（原卷 13–15 頁）需向照片來源或同學取得。推測單元是 Chap.4「Architecture Improvement of Timing, Area, and Power」（pipeline、retiming、parallel、area/power）：其餘題目已對應 Chap.1–3、5、6，只剩 Chap.4 沒出現；依據是 115-1 講義，2023 年版章節可能不同。
- 下一步（使用者要求）：再做一題 STA，並補 Problem 5。已在 Google 日曆建提醒（2026-11-04 10:00 Asia/Taipei）。
- STA 常搞混：Tco（clock→Q，加在 arrival）與 Tsetup（加在 required 減項）在本題都是 0.4；題目沒給 clock uncertainty。
