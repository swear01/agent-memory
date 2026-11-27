---
title: CVSD 期中考古題準備（2023 fall 題目重打版）
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-11-04
---

# CVSD 期中考古題準備
- 學習用副本：`<GoogleDrive-root>/document/學習紀錄/碩一/電腦輔助積體電路設計/midterm-prep/CVSD_2023_fall_midterm_questions.pdf`（15 頁，只有題目，無手寫答案或批改）。該資料夾 README.md 已加一行索引。
- 同檔另放講義區：`<GoogleDrive-root>/document/學校講義/碩一/電腦輔助積體電路系統設計/考古/CVSD_2023_fall_題目.pdf`（與 `CVSD_2015.pdf` 並列）。
- DriveFS 的 `com.google.drivefs.item-id#S` 缺失時另驗遠端；`touch` 和刪 quarantine 曾無效。已用相同內容的新檔取代卡住的本機副本，核取得 item-id，動作前核既有遠端檔及來源。相關 [[tools/docx/word-layout-and-drivefs]]。
- 用歷屆原題，不自編。Drive 同目錄的 `CVSD_歷屆原題_P5架構+STA.pdf` 有 2023 Spring P5 Architecture Improvement、2018 Spring P4 retiming/skew、2022 Fall V STA。2023 Fall 原照片缺封面、P5 (p13–15)；SDF 末兩行截斷，重打標 `...`。P1/P3 部分波形由程式推回、buf/inv 數須查原照。
- 歷屆無文字層的掃描 PDF 用 `pdftoppm` → `tesseract` 才 grep。AI 擦手寫會破壞印刷程式碼，用 HTML 重打、inline SVG 重畫，再跑 `/Applications/Chromium.app/Contents/MacOS/Chromium --headless=new --no-pdf-header-footer --print-to-pdf=<out.pdf> <in.html>`。

## 解題慣例（以原卷批改為準）

- STA：輸入/輸出延遲相對理想 CLK；Tclkx 是 clock latency，兩顆暫存器相同。暫存器端 setup required = T + Tclkx − Tsu，hold required = Tclkx + Th；輸出端 setup required = T − DO，hold required = −DO；暫存器起點 arrival 從 Tclkx + Tco 起算。
- SDF：IOPATH 兩組括號是 output 的 rise/fall（不是 input），每組是 (min:typ:max)；max delay 取第三個，fast corner 取第一個。
## Chap.4 解題慣例（115-1 Chap.4 投影片 p.6、p.10–15）

- Latency（cycles）= 路徑上的 register 數；ns = cycles × T。Parallel 的 latency 不變。
- Throughput：OPs/cycle = 路徑上各運算數相加；MOPS = OPs/cycle × f；Mb/s = 輸出 bit 數 × f（parallel 再乘路數）。
- Overhead：pipeline 多 register 面積、latency、每級 Tco + Tsetup、切不平均；parallel 約 2 倍面積與功耗。
- Setup：T ≥ tpcq + tpd,max + Tsetup。Hold：tccq + tcd,min ≥ Thold；hold 和 T 無關，不能靠降頻修，要加 buffer。
- Capture 端 clock 晚到 δ：setup 變 T + δ ≥ …，hold 變 … ≥ Thold + δ。正 skew 對 setup 有利、對 hold 不利。
- Retiming：register 往輸入側移過 n 輸入元件會變 n 顆；目標是讓各級 tpd 平衡（2018 Spring P4：5/1 → 3/3，同時修好 setup 與 hold）。
- 易混觀念：fork 與 begin…end 的延遲累加、未標寬常數 32-bit signed、一般低溫較快（忽略 temperature inversion）、DesignWare area/speed trade-off、WLM 估 interconnect area。
