---
title: "子任務 usage-limit 錯誤不能當成研究結果"
scope: "tools/agent-collaboration"
status: active
updated: 2026-09-07
evidence_digest: 97f9d32e8af9bbf47befd8ba97a8ddb5c33aebec79cf8eccef0c92d09c63472e
---

# 子任務 usage-limit 錯誤不能當成研究結果

## Problem

歷史子任務通知明確為 errored，原因是 usage limit，沒有交付有效結果。

## Durable rule

將子任務成功結果與錯誤通知分開；保留未完成範圍並據實報告。這份證據不能證明提高 effort 就是額度耗盡的原因，也不能从 credits 欄位推斷可用配額。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
