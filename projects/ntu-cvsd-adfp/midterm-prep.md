---
title: CVSD 期中考古題準備（2023 fall 題目重打版）
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-11-04
---

# CVSD 期中考古題準備

期中考已於 2026-11-04 考完；本頁留作期末與日後複習參考。

## 成果位置（2026-11-03 驗證）

- 學習用副本：`<GoogleDrive-root>/document/學習紀錄/碩一/電腦輔助積體電路設計/midterm-prep/CVSD_2023_fall_midterm_questions.pdf`（15 頁，只有題目，無手寫答案或批改）。該資料夾 README.md 已加一行索引。
- 同檔另放講義區：`<GoogleDrive-root>/document/學校講義/碩一/電腦輔助積體電路系統設計/考古/CVSD_2023_fall_題目.pdf`（與 `CVSD_2015.pdf` 並列）。
- `<GoogleDrive-root>` = 使用者指定帳號的 DriveFS「我的雲端硬碟」。兩份用 `cmp` 核對本機逐位元組相同。2026-11-04 查到這兩份一直沒有 `com.google.drivefs.item-id#S`（沒上傳），`touch`、刪 quarantine 都無效；複製成新檔、刪原檔、改回原名後幾十秒內取得 item-id。練習卷 `CVSD_歷屆原題_P5架構+STA.pdf` 原本就有 item-id。

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
- 待補：Problem 5（原卷 13–15 頁）仍缺原題。已確認單元是 Chap.4：2023 Spring 考卷 Problem 5 就叫 "Architecture Improvement"（8pts，pipelining/parallelizing 的 latency、throughput 填表 + overhead 簡答）；2018 Spring 也考過 retiming 與 skew 不等式。
- 歷屆考古掃描檔在 `~/Documents/Codex/2026-11-02/new-chat-2/work/cvsd-exams/`（2020–2023 與 older/2013–2019），PDF 無文字層，要 `pdftoppm` + `tesseract` OCR 後才能 grep。STA 題型參考 2022 Fall p17–18、2023 Spring p19。
- 練習題一律用歷屆原題，不要自編（使用者 2026-11-03 明確要求）。已重打（去手寫答案、圖重畫）：2023 Spring P5 Architecture Improvement、2018 Spring P4 Important Timing Parameters（retiming/skew）、2022 Fall Problem V STA（28pts，參數 Tco 0.5、Tclk 0.3、Tsu 0.4、Th 0.1、DA=DB=DC=2.5、100MHz）→ Drive `midterm-prep/CVSD_歷屆原題_P5架構+STA.pdf`。產生器在 session scratchpad（build.py + gen/gen.py 的 STA2 時序圖）。2023 Spring P8 STA 電路與 2023 Fall P7 幾乎相同，不必重做。
- 考前（2026-11-04）走完 2023 Spring P5 與 2018 Spring P4；2022 Fall STA 沒做到。
- STA 常搞混：Tco（clock→Q，加在 arrival）與 Tsetup（加在 required 減項）在本題都是 0.4；題目沒給 clock uncertainty。

## Chap.4 解題慣例（115-1 Chap.4 投影片 p.6、p.10–15）

- Latency（cycles）= 路徑上的 register 數；ns = cycles × T。Parallel 的 latency 不變。
- Throughput：OPs/cycle = 路徑上各運算數相加；MOPS = OPs/cycle × f；Mb/s = 輸出 bit 數 × f（parallel 再乘路數）。
- Overhead：pipeline 多 register 面積、latency、每級 Tco + Tsetup、切不平均；parallel 約 2 倍面積與功耗。
- Setup：T ≥ tpcq + tpd,max + Tsetup。Hold：tccq + tcd,min ≥ Thold；hold 和 T 無關，不能靠降頻修，要加 buffer。
- Capture 端 clock 晚到 δ：setup 變 T + δ ≥ …，hold 變 … ≥ Thold + δ。正 skew 對 setup 有利、對 hold 不利。
- Retiming：register 往輸入側移過 n 輸入元件會變 n 顆；目標是讓各級 tpd 平衡（2018 Spring P4：5/1 → 3/3，同時修好 setup 與 hold）。
