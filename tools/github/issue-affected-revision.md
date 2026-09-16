---
title: "報告 issue 前先確認缺陷存在於哪個 revision"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: 89f11f3ee5ff3898e87791aa7082fa2acaa4d25d46370385316c443c8d5883ff
---

# 報告 issue 前先確認缺陷存在於哪個 revision

歷史助手把一個仍未合併 PR 的 MCP elicitation 行為寫成 issue 根因；使用者指出 main 上根本沒有該變更，要求回到原 PR 修正並補測試。

將觀察用的分支、commit、安裝版與 main 分開，先確認缺陷所在版本再選擇回報或修補位置。候選 PR 的設計問題不能直接描述成正式版本回歸。來源沒有後續修補與測試結果；其具體 approval 政策也不能由這份歷史摘要自動採用。
