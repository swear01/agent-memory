---
title: "修正共用終端機邏輯時檢查所有使用者"
scope: "projects/magic-storage"
status: active
updated: 2026-09-07
evidence_digest: 971b416f50f616b21403221cecb72c3fb02ddba736fe920d907e459016b99baf
negative_result: false
redaction: passed
---

# 修正共用終端機邏輯時檢查所有使用者

## Problem

Agent 只修正 crafting terminal，漏掉應共用行為的 storage terminal。

## Mechanism

以單一畫面為修正範圍，沒有追查共同邏輯的其他使用者。

## Durable rule

先追查共用終端機邏輯的所有使用者，在共同位置修正；驗證 crafting 與 storage terminal 都得到相同修正。

## Boundary

適用於此專案共享終端機行為；storage terminal 是功能較少的版本，各自特有功能仍應分別驗證。

## Verification

來源使用者直接指出只修了一個 terminal，並明確要求共用程式碼；來源片段未提供最終修正通過的證據。
