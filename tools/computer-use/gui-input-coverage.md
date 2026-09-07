---
title: "鍵盤與截圖成功不代表滑鼠互動已驗證"
scope: "tools/computer-use"
status: active
updated: 2026-09-07
evidence_digest: 92b50304489bf2f67460ff07c97bdd95c9cece1c16e11a4e01ce0b214aebfd96
---

# 鍵盤與截圖成功不代表滑鼠互動已驗證

## Problem

Agent 回報 macOS 合成 mouse-down/up 被 LWJGL 忽略，native pipe 回報 sender unauthenticated；GUI shift-click 因此未完成。

## Durable rule

依互動類型分別記錄驗證結果。保留失敗操作與未完成的目視風險，不能以其他輸入或 GameTest 成功代替 GUI click 驗證。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
