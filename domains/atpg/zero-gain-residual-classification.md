---
title: "ATPG 零新增偵測不等於所有 residual 都不可測"
scope: "domains/atpg"
status: active
updated: 2026-09-09
evidence_digest: 39068903ad7fc5f299fe86f899d48ed415297c69d3200a2a8d09d9b9d2032cb4
---

# ATPG 零新增偵測不等於所有 residual 都不可測

## Problem

歷史 progressive residual CSV 中，b03、b04、b05、b07、b08、b09、b13 的 T2 與 T4 新增偵測皆為零。模型草稿卻把它推論成剩餘故障都是 structurally untestable。

## Durable rule

保留指定電路與該輪設定下的零增益結果，分開核對 residual 數量與 AU、AB、TO 等分類；不可由一次零增益直接宣稱所有剩餘故障不可測或更長 frame 永遠無效。

## Boundary

控制者解析了 7 列原始 CSV：每列 R1_count 都大於 T1_AU，例如 b03 是 88 對 24。這已否定「全部 residual 等於 AU」的草稿說法；未重新跑 ATPG，也未證明其餘 residual 的可測性。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
