---
title: "約束後的證明須回到原始模型檢查"
scope: "projects/pono-llm"
status: active
updated: 2026-09-08
evidence_digest: 7f06d33ac024957384adb4484d0df2a70eb61bf9bfcf6892db607a2b60b5569d
---

# 約束後的證明須回到原始模型檢查

## Problem

來源表格中三個候選 certificate 在原始 transition 的 C2 檢查遭拒絕，另三個案例因 signal mapping 缺失而未完成；先前加 hint constraint 的 UNSAT 結果不足以驗證這些 certificate。

## Durable rule

對原始模型核對 Init 蘊含 Inv、Inv 與 Trans 蘊含下一步 Inv，以及 Inv 蘊含安全性；不要用同一個尚未證實的 hint 約束去驗證候選 invariant。明確區分 certificate 被拒絕、mapping 未解決與原始模型是否安全。

## Boundary

來源直接列出三個 C2=SAT 的拒絕結果，不能外推到全部四十二個案例。候選 invariant 不歸納不代表原電路不安全；另經證明有效的約束也不能一概判為不合法。本次未重新執行 checker。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
