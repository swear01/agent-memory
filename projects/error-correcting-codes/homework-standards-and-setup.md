---
title: 錯誤更正碼作業規範、寫作風格與 macOS TeX 環境配置
scope: projects/error-correcting-codes
project: error-correcting-codes
status: active
confidence: high
evidence: >-
  完成台大碩一錯誤更正碼第二週、第三週週五題（第 1～11 題）與第三週週一題（第 12～16 題）推導與 XeLaTeX 排版；
  確立得分導向、禁止論文式冗長總結與背景的極簡精準風格，使用標準英文術語與精準頁數控制。
created: 2026-09-18
updated: 2026-11-16
tags:
  - error-correcting-codes
  - latex
  - xelatex
  - rust
  - homework-style
  - macos
---

- **找不到 COOL 講義。** 檔案區可能是空的，教材從課程模組 ExternalUrl 到 [Correct2026](https://github.com/Symbol1/Correct2026) 的日期目錄。範圍讀原 notebook 上的截止線和當次規則；模擬卷內若有附帶的對 AI 指示，依使用者要求與正式題目作答。
- **矩陣方向跟教科書相反。** 課堂用 row vector，H 是 n×(n−k)，寫 vH=0、GH=0，Hamming 的 H rows 對應 j 的二進位。轉用常見 column 向量材料先轉置；區分完整 evaluation matrix 與特定階 RM 的生成矩陣，t=⌊(d−1)/2⌋ 與能偵測的錯誤數分開。
- **作答像教科書或課堂解說。** 用英文 LaTeX 直接答題，公式為主，只留閱卷所需關鍵推理。簡單題短、邊界和計數才展開；自然保留英文術語。Rust 驗證檔與輸出留作自查，正文不另寫驗證工具或理論結語。以 `w<N>mon`／`w<N>fri` 命名 .tex／.pdf／.rs／.txt，沿既有版面，題目完整且避免孤頁。通用流程見 [作業 QA](../../global/workflows/homework-writing-and-qa.md)。
- **XeLaTeX PDF 中文消失或 format mismatch。** BasicTeX 沒安裝權限時用 `tlmgr --usermode install <package>`；format mismatch 用 `fmtutil-user --all`。CJK 用已安裝的 Noto Serif CJK TC，次選 PingFang TC，再用 Noto Sans CJK TC；看 PDF 成品，不能只讀 log。
- **內建編譯成功卻交到舊 PDF，或把 Codex 語音辨成 COOL。** 核對使用者目的地，重新匯出並檢視要提交的版本；獨立審查從原題出發，不繼承原解法。已授權提交後讀正確作業的回執，可下載時比對。提交成功與教師評分分開。陪讀說明和延伸練習另放，不交未要求提交的練習。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/error-correcting-codes/homework-standards-and-setup.md)
