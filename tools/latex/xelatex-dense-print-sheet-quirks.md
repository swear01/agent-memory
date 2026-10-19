---
title: XeLaTeX 高密度黑白參考資料的排版陷阱
scope: tools/latex
tool: xelatex
status: active
created: 2026-10-19
updated: 2026-10-19
tags:
  - xelatex
  - xecjk
  - tabularx
  - multicol
---

# XeLaTeX 高密度黑白參考資料的排版陷阱

2026-10-19 在 SwairM5（BasicTeX 2026）排 A4 雙欄、5 mm 邊距的考試參考資料時實測。

## xeCJK 的 Noto Sans CJK TC 粗體會抓成 Black

只寫 `\setCJKmainfont{Noto Sans CJK TC}` 時，`\bfseries` 的中文落在 `NotoSansCJKtc-Black`，印出來過粗。要明確指定：

```latex
\setCJKmainfont{Noto Sans CJK TC}[BoldFont={Noto Sans CJK TC Bold}]
```

用 `pdffonts` 確認嵌入的是 `NotoSansCJKtc-Bold`，不要只看 PDF 有產生。

## `tabularx` 不能包在 `\newcommand` 裡

把 `\begin{tabularx}...` 放進自訂巨集（例如 `\newcommand{\tx}[1]{\begin{tabularx}{\linewidth}{#1}}`）會在編譯時報：

```text
! File ended while scanning use of \TX@get@body.
```

`tabularx` 要讀環境原文，只能直接寫 `\begin{tabularx}...\end{tabularx}`，或用 `\newenvironment` 之外的其他方式。

## 長表格在 `multicols` 裡造成大塊空白

`tabularx` 不能跨欄，一個長的定義表會被整塊推到下一欄或下一頁，留下空洞。名詞類內容改成一行一條的懸掛縮排段落即可自然斷欄：

```latex
\newcommand{\term}[2]{\par\noindent\hangindent=1.2em\hangafter=1{\bfseries #1}\enspace #2\par}
```

另外 `\vspace{0pt plus 4\baselineskip}\penalty-150\vspace{0pt plus -4\baselineskip}` 這種「標題前預留空間」在 `\raggedcolumns` 下會讓欄底空白變多；高密度版面直接用 `\par` 比較好。

## 符號缺字

Latin Modern 沒有 `⇒ ⇔ ≡ ①` 等 Unicode 符號，會出現 `Missing character`。兩種解法擇一：換成 `unicode-math` + `STIX Two Math`，或把該區段交給 CJK 字型：

```latex
\xeCJKDeclareCharClass{CJK}{"2190 -> "22FF, "2460 -> "24FF, "25A0 -> "25FF}
```

## 黑白列印

- 印表機若印黑白，色塊標題、灰字都會變淡或糊掉；層級改用粗字 + 黑線、黑底白字方塊（`\colorbox{black}{\color{white}...}`）表現。
- 細襯線字（Latin Modern）黑白影印偏淡；Arial + Noto Sans CJK TC + STIX Two Math 筆畫較粗。

## 版面收尾

- 用 `pdfinfo` 數頁、`grep -cE 'Overfull \\hbox|Missing character'` 查 log、`pdftoppm` 轉圖逐頁看，三者都要做。
- 頁數超出時先刪重複與不必要的內容，再微調字級／行距；在固定頁數內可用迴圈試行距，取「不增頁且無 overfull」的最大值。
