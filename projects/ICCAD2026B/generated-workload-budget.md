---
title: "產生資料後核對總工作量，不能只看 validator 通過"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-07
evidence_digest: 5f31f0e199f847fa954cfd08993d35856ffcf13564a0fda0f236df5a6f8eb24b
---

# 產生資料後核對總工作量，不能只看 validator 通過

## Problem

歷史 generator 記錄指出目標約 10M trace rows 的資料集只有約 122K rows。

## Durable rule

核對產生後的總行數與預定工作量，對 base range 設定不足給出明確失敗；一般格式檢查通過不代表 benchmark 規模正確。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
