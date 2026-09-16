---
title: "Build 失敗不能留下預先增加的部署版本"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 3d231cb9b74e165cd7c0fa0771eee8c26cb8b0eb0e1daea71e66e3a5eb4f49d1
---

# Build 失敗不能留下預先增加的部署版本

歷史助手發現部署腳本先 bump 版本才 build，失敗後版本號仍增加，再重試又會增加；新增測試也因版本未還原而失敗。

將建置前版本變更與失敗處理視為同一交易，在 build 或產物驗證失敗時保留原版本，並核對重試不會額外累加。回寫前仍須確認沒有其他作者的並行修改。來源只到 RED 與最小 rollback 計畫，沒有修補後通過證據。
