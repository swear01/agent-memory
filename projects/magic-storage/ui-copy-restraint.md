---
title: "玩家介面避免反覆加入多餘解釋"
scope: "projects/magic-storage"
status: active
updated: 2026-09-07
evidence_digest: 3ce49ceb83a9ebd20bddac418d76084a3cfa76660fb160febac1c7baf612005b
negative_result: false
redaction: passed
---

# 玩家介面避免反覆加入多餘解釋

## Problem

Agent 多次在玩家介面加入多餘操作提示與機制說明，使用者再次要求移除。

## Mechanism

把開發者想交代的實作細節放進玩家介面，增加了不必要的標籤。

## Durable rule

玩家介面以辨識與操作必要資訊為主；新增提示前檢查它是否幫助玩家作決定，避免重加使用者已要求移除的說明。

## Boundary

這是此遊戲專案的玩家介面偏好，不代表刪除必要錯誤訊息或無障礙說明。

## Verification

來源含使用者對 energy/t 說明與前後切換提示的具體糾正，並要求將 simple is better 寫入專案規則；本次沒有重跑 GUI。
