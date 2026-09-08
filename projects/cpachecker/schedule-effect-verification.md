---
title: "LLM 排程單元測試不能代替效果驗證"
scope: "projects/cpachecker"
status: active
updated: 2026-09-08
evidence_digest: d706af73e0699fe46642f7bec8313933d717b51d126bf45f5550412ccadea445
---

# LLM 排程單元測試不能代替效果驗證

## Problem

Agent 實作 EVERY_N_OR_INTERVAL 後，以增量編譯、排程測試 8/8、選項測試 2/2 作為階段結果，尚未跑 benchmark 就詢問下一步。使用者明確要求繼續做到完整 dataset 的效果驗證。

## Durable rule

在已授權的排程改善工作中，完成觸發邏輯測試後繼續實際資料集驗證，分別報告單元測試、完整建置與 benchmark 效果；遇到真實阻礙才回報阻礙，不能把中途詢問當成完成。

## Boundary

本條保留該次使用者糾正與驗證層次。歷史片段沒有完整 dataset 結果，亦未證明 wall-clock 到時必定獨立觸發或排程改動已改善效能。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
