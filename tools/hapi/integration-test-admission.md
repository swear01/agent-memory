---
title: "整合測試不要因本機 hub 剛好在線而自動啟用"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 1df0936877794920c9b8f5e08044729f65cf4fb2a1ddbd954286630d02ce265f
---

# 整合測試不要因本機 hub 剛好在線而自動啟用

歷史助手回報一般測試因偵測到本機 hub 可用而跑進只適合 integration 環境的 suite，產生競態失敗，並提出明確的 integration 啟用條件。

將測試環境的明確選擇與依賴是否偶然可達分開。Integration admission 應指向隔離資源；一般套件略過它時也要說明覆蓋邊界，不能稱整合測試已通過。來源說重跑仍在進行，沒有最終綠燈證據。
