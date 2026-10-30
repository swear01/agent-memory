---
title: PDF 文字抽取會把底線變空白：識別字一律對照渲染頁面
scope: tools/documents
status: active
updated: 2026-10-31
tags:
  - pdf
  - pdftotext
  - spec-reading
  - LSV
---

# PDF 文字抽取會把底線變空白

LaTeX 排版的題目 PDF（`\_`、`\texttt{a\_b}`）經 `pdftotext` 類工具或課程教材的 `*.pages.md` 逐頁抽取後，底線常被抽成空白。LSV PA1 實例：原 PDF 寫 `lsv_cut_tt <k>`，抽取文字變成 `lsv cut tt <k>`；照抽取文字實作成 `lsv` + 子命令，24 個測資全 AC 仍被判 alt syntax，Exercise 4 從 80 分掉到 48 分（見 `projects/LSV-PA/pa1-2026.md`）。

## 警訊：看到就要停下來核對原 PDF

同一份抽取文字裡出現下列「明顯少了底線」的字，就代表整份文件的底線都不可信：

- C 型別或 API：`uint64 t`（應為 `uint64_t`）、`Cudd DagSize`（應為 `Cudd_DagSize`）、`size t`。
- 工具指令：`print stats`、`show bdd`（ABC 實際是 `print_stats`、`show_bdd`）。
- 任何「指令名 + 參數」格式裡多出看似子命令的空白單字。

只要有一個警訊，題目要求的指令名、函式名、檔名、輸出格式都要用 `pdftoppm -f <頁> -l <頁> -r 80 -png` 渲染該頁，看圖確認，不可只信文字。

## 跨 note 同步

2026-09-29 另一個 session 已在 `projects/ntu-lsv-course/bilingual-video-workflow.md` 記下這個底線問題，但沒有同步到正在進行的 `projects/LSV-PA` note，也沒有通知使用者；Exercise 4 截止日是 10/9，原本有十天可以修。發現會影響其他進行中作業／專案的事實時，要同時更新那個專案的 note，並在回覆中明確告訴使用者。開始做作業前，要搜尋整個 memory（例如 `qmd search "pages.md 底線"`），不能只讀該專案目錄。

## 根因與全面檢查（2026-10-31）

LaTeX 在非等寬字型裡的 `\_` 是畫出來的細線段，不是字元，所以連 `pdftotext -layout` 也抽不到，換抽取工具沒用。可靠做法是用 `pdfplumber` 讀 `page.rects + page.lines`，找高度 ≤0.8pt、寬 1–7pt、落在文字基線上的線段，再取左右緊鄰的字，就知道哪兩個字之間少了 `_`。PowerPoint 匯出的投影片底線是真字元，抽取不會掉。

LSV 2026 課程教材 16 份 PDF 全掃：只有 `LSV_2026_pa1`（46 處）、`hw2`（`value@decision_level`）、`ls-handout`（`CHOOSE_DIVISOR`）真的掉了底線，已在對應的 `*.pages.md` 補回並逐行 diff 確認沒有其他改動。`lecture02-ba` 第 27 頁命中的是數學式 `{0,1}` 下方的向量短橫，屬於誤判。`hw3.pdf` 沒有命中。偵測與修補腳本放在課程教材 `影片解析/資料說明/find-drawn-underscores.py`、`restore-drawn-underscores.py`；之後新的 HW／PA PDF 先跑偵測腳本，再看渲染頁面確認。
