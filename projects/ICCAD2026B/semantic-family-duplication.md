---
title: "不同指令或 ELF 不一定是獨立失敗家族"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-07
evidence_digest: 54fcba9906f688105fb283ab49d69e5e8be080cffe1c1c59c0dbc2b71be4ab5e
---

# 不同指令或 ELF 不一定是獨立失敗家族

## Problem

兩個自相依指令 body 被回報可觸發相同 stale IF-wait 機制；其中一個的語意分離仍是條件性假設。

## Durable rule

家族去重應比較因果事件、失敗機制與 oracle；不能只因 body 指令或雜湊不同就增加獨立家族計數。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
