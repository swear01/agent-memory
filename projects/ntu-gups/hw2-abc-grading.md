---
title: GUPS3006 EDA（IEDA）HW2 Berkeley ABC 批改：流程、TA 給分決定、舊版 ABC 陷阱
scope: projects/ntu-gups
status: done
updated: 2026-11-10
tags:
  - ntu-gups
  - ieda
  - abc
  - grading
  - ntu-cool
---

# GUPS3006 EDA（IEDA）HW2 批改（2026-11-10 完成）

COOL course `64904`、assignment `422755`（Logic synthesis and verification with Berkeley ABC）。使用者是助教。
7 份已評分寫入 COOL；2026-11-10 老師同意後已公布（HW2 與期中 Midterm Score 皆 7/7 posted），並發公告 discussion_topic 545830。HW1（417944）仍未公布。COOL 名單上未繳的那位是**旁聽生，不用給分**。
成績與評分紀錄在 mazu `~/IEDA/HW2/reference/`（`RUBRIC.md`、`grades/summary.csv`、`grades/<學號>.md`），不放本庫。

## 取得資料（快路徑）

COOL 存取與寫分方式見 `domains/other/ntu-cool-api-access.md`：Mac 用 `~/IEDA/tools/cool.sh`（Brave 分頁 XHR）下載所有繳交，`rsync -a ~/IEDA/ mazu:IEDA/`，批改在 mazu 做。HAPI peer 一律 `--permission-mode auto`（見 `tools/hapi/session-permission-preference.md`）。

## TA 給分決定（可沿用到之後的 ABC 作業）

- Part 3–9 各 10 分，c432 優化 30 分。c432 以 strash 後 `and =` 計：≤119 滿分、121 → 80%、137 → 50%，中間線性。
- **分數一律整數**（四捨五入）。
- 只有圖、沒有 `print_stats` 輸出：該 part 扣 2。有檔案或圖可證明做過就只扣這 2 分；真正缺結果才多扣。
- 沒有獨立的「書面報告」分數；題目 Delivery 只要求各 part 結果與 c432 優化。
- **流程比數字重要**：因 ABC 版本造成的數字差異不扣分。

## 技術發現

- c432（題目附的 Genus 版 `c432.v`，36/7，strash 208）：`&deepsyn`、`&transduction` 多種子全部收斂在 **119**，`cec` 等價；網路與論文找不到更小的公開數字。照題目 Part 7 流程是 137。
- 題目寫的 `read_liberty` 在新版 ABC 已不存在，要用 `read_lib`。Part 9 `read_verilog -m c432_syn.v` 前必須先載入元件庫，否則閘被當黑盒、`cec` 無效。
- **Part 8 出現 181 閘／面積 331 = 官網舊版 Windows 執行檔 `abc10216.exe`**（md5 與 alanmi 網站相同；新版是 173／315）。舊版 `strash; resyn2` 只到 150（新版 137），Part 7 先 `fraig` 的流程新舊都到 137。已排除元件庫轉換、supergate、指令順序、32/64 位元。git 歷史 2009-05 到 2010-11 沒有 commit，編出的鄰近舊版得 175／321，未能逐字重現 181；舊版讀這份 Genus `c432.v` 會 segfault，要先用新版轉 BLIF。mazu 無 wine。
- `&rewire` 對 c432 會因 36 變數真值表配置爆記憶體而 segfault。
- `&r` 讀回 AIGER 會失去 PI 名稱，直接 `cec c432.v` 會失敗；改與已驗證的 AIG 比。
