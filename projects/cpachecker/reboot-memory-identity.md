---
title: "重開機後的可用記憶體不能當固定機器身分"
scope: "projects/cpachecker"
status: active
updated: 2026-09-07
evidence_digest: 541719673fd046c5f7df683d9317c9442aac472bbdf3552ce7b6e41363e7e962
---

# 重開機後的可用記憶體不能當固定機器身分

## Problem

歷史恢復程序因 MemTotal 相差 12,288 bytes 而拒絕同一台主機。

## Durable rule

固定機器身分應使用穩定欄位；重開機相關數值應另行驗證，仍保留其他身分與中斷證據檢查。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
