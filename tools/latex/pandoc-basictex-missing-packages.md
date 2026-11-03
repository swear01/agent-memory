---
title: pandoc 用 BasicTeX 排中文履歷時缺 titlesec／enumitem
scope: tools/latex
tool: pandoc
status: verified
created: 2026-11-03
updated: 2026-11-03
tags:
  - pandoc
  - xelatex
  - basictex
  - cjk
---

# pandoc 用 BasicTeX 排中文履歷時缺 titlesec／enumitem

2026-11-03 在 macOS（BasicTeX）以 `pandoc --pdf-engine=xelatex` 把 Markdown 履歷排成 A4 時實測。

## 現象

用 `-H header.tex` 想壓縮標題與清單間距，`\usepackage{titlesec}`、`\usepackage{enumitem}` 都報：

```text
! LaTeX Error: File `titlesec.sty' not found.
! LaTeX Error: File `enumitem.sty' not found.
```

pandoc 失敗時舊 PDF 仍留在原處，`pdfinfo` 看到的頁數可能是上一版，要先看 pandoc 的 stderr。

## 不裝套件的做法

只靠 LaTeX 核心巨集就夠把兩頁壓回一頁：

```latex
\makeatletter
\renewcommand\section{\@startsection{section}{1}{\z@}{-1.6ex}{0.6ex}{\normalfont\Large\bfseries}}
\renewcommand\subsection{\@startsection{subsection}{2}{\z@}{-1.4ex}{0.5ex}{\normalfont\large\bfseries}}
\makeatother
\setlength{\parskip}{3pt}
```

`-H` 內容排在 pandoc 模板設定 `\parskip` 之後，所以覆寫有效；實測影響最大的是 `\parskip`，單改它就從兩頁變一頁。中文字型用 `-V CJKmainfont="Noto Sans CJK TC" -V geometry:a4paper,margin=1.5cm`。

## 檢查換行

```bash
pdftotext -layout out.pdf - | grep -v '^\s*$' \
  | awk 'prev ~ /・/ && $0 ~ /^ {3,}[^ ・]/ {print "WRAP: " prev} {prev=$0}'
```

可列出條列項目被折成兩行的地方，逐條縮字到一行。
