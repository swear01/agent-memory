---
title: "暫存工作須核對實際共用分割區與保留證據"
scope: "machines/hapi-fleet"
status: active
updated: 2026-09-07
evidence_digest: 52dc6fdbc7c71281f1c632fb021f676f095ac00b454b6df934c48e0cc8fe4df5
---

# 暫存工作須核對實際共用分割區與保留證據

## Problem

歷史檢查顯示 /tmp 與 /var/tmp 共用已滿的根分割區，大量暫存工作是主要占用。

## Durable rule

以檔案系統容量與來源清單判斷占用；工作結束應有可審查的暫存清理範圍，保留運行中工作與必要證據。不能把目錄名稱當獨立容量或刪除授權。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
