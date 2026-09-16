---
title: "字型查找失敗與系統字型依賴要分開處理"
scope: tools/latex
status: active
updated: 2026-09-16
evidence_digest: 883f714ea2bd0aed4875fea0951cfa4f21427de94bdf0949e54fe7648f6cba14
---

# 字型查找失敗與系統字型依賴要分開處理

原始編譯 log 有多個系統絕對路徑的可重現性警告，另有 Fira 缺失警告；真正停止點是 fontspec 找不到指定的 Noto Sans CJK TC Medium，結果沒有頁面輸出。

先辨識 fatal lookup error，核對實際 family／style 名稱與建置字型來源，再驗證成品。系統絕對路徑代表環境依賴，不自動證明所有環境都不可重現；修正名稱或安裝字型也不保證字形覆蓋與版面正確。來源没有修復後結果。
