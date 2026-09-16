---
title: "編輯投影片前確認實際編譯入口"
scope: tools/documents
status: active
updated: 2026-09-16
evidence_digest: 599f58d8d084403f8926b52d844452ca829d1e15d7f737da62d943f3877a7395
---

# 編輯投影片前確認實際編譯入口

歷史助手發現 main.tex 實際編譯的投影片檔名，與一直編輯的檔案完全不同，因而回頭檢查真正使用的來源。

沿建置入口與 include/input 關係確認目標檔確實進入交付物，再修改並檢查新輸出。編輯成功不代表改動會出現在 PDF；來源只到檔名錯配的發現，沒有更正後編譯或視覺驗收結果。
