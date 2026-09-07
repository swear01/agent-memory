---
title: "監控排程必須驗證下一批接續"
scope: "tools/agent-collaboration"
status: active
updated: 2026-09-08
evidence_digest: 95ceb4ec47df68ac04d9ae6de4210068d005ecb073fdb8af1b4a0d768fa15c0b
---

# 監控排程必須驗證下一批接續

## Problem

歷史助手承認 timer 只監控狀態，上一 shard 結束後未啟動下一批，導致整夜閒置。

## Durable rule

需要自動接續的工作，驗證必須跨越上一批完成、產物檢查、容量判斷與下一批實際啟動。若容量不足，保留明確等待原因與下一次檢查；timer 活著或狀態更新不算接續成功。

## Boundary

適用於分批 benchmark 與背景佇列。來源只報告 scheduler 測試通過，當時仍無可用容量，不能宣稱實際 handoff 已驗證。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
