---
title: "計數器 oracle 要計入觀察指令本身的退休成本"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-07
evidence_digest: a8722438514cdb7dd50a5924c6042552ae363bdd4433c9f338646a67fd855910
---

# 計數器 oracle 要計入觀察指令本身的退休成本

## Problem

歷史 pilot ledger 寫明硬編碼的低半部期望值漏算中間 li 退休，導致 reference 與 buggy 都失敗。

## Durable rule

以觀察 anchor 與明確退休指令數推導期望值；先修正 oracle 的基準錯誤再解讀成對差異。來源中的 retry 仍只屬 pilot，不能當正式收集或發布完成。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
