---
title: "回答轉換必須符合各 provider 的問題身份契約"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 2e21c4a3f8ef7f2ca392a3950f831fe9c7c1fcd9f19f14b56f87354d8827dc3e
---

# 回答轉換必須符合各 provider 的問題身份契約

保存程式按 question 文字建立 Claude answers，空字串直接跳過；輸出案例顯示使用者即使已選擇也變成沒有回答，多題時可能只留下部分。Cursor 的 question／option ID 與 Codex 的 nested answers 又是不同格式。

在 provider 邊界核對實際 schema 與答案身份，不能共用錯誤的 index／文字假設；缺欄位須顯式處理而非靜默丟回答。來源的重現是合成輸入，尚在查真實 Claude 是否產生空 question；因此可確認 mapper 丟資料，不能確認生產 hang 的觸發頻率或修復完成。
