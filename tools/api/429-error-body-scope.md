---
title: "429 診斷需保留錯誤內文與驗證範圍"
scope: "tools/api"
status: active
updated: 2026-09-09
evidence_digest: ae746da94c7c71e10d6009cabfaf25e2a92f8c012ba2c4cf1b16657e802ffae0
---

# 429 診斷需保留錯誤內文與驗證範圍

## Problem

歷史 embedding 請求的原始 429 RESOURCE_EXHAUSTED 內文明示 prepayment credits are depleted；assistant 隨後擴張為所有 code/config 都沒問題，並保證補額度後兩條路徑都能用。

## Durable rule

診斷 429 時讀取實際錯誤內文與請求路徑，將此案例記為 embedding 請求遭額度阻擋。解除阻擋後仍需重新驗證，不能由一次錯誤保證其餘設定正確或恢復成功。

## Boundary

此片段只有 embedding 的原始錯誤；completion 結果僅有 assistant 摘要。沒有補額度後的成功請求，也不支持目前模型免費額度、價格或方案的任何結論。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
