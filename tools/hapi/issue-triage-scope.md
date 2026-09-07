---
title: "使用者要求全量 issue 審查時不可只看子集"
scope: "tools/hapi"
status: active
updated: 2026-09-07
evidence_digest: 87557fa5031ecf9e9e5f2e97ffd39ec7758bd0fcee7554a3808b5cfa92624b67
---

# 使用者要求全量 issue 審查時不可只看子集

## Problem

歷史 HAPI 工作中使用者糾正 issue 涵蓋太窄，要求所有 open issues 都看過，並追問是否實機驗證。

## Durable rule

在明確要求全量審查的任務中先核對完整清單與涵蓋範圍；被要求實機 e2e 時完成該驗證再宣稱完成。歷史 bot 追蹤要求不得被擴大為所有未來 PR 的固定政策。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
