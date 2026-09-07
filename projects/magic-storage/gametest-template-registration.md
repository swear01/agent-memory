---
title: "新增 GameTest 類別需有有效 template"
scope: "projects/magic-storage"
status: active
updated: 2026-09-07
evidence_digest: 8addbb46757135e89e755a4c6a365b4546669917d2c682432c86faba5229aafe
---

# 新增 GameTest 類別需有有效 template

## Problem

Agent 記錄新增測試類因沒有 template 而使 GameTest 啟動崩潰，隨後移到既有 platform 測試類再跑。

## Durable rule

新增測試時核對 framework template 綁定，優先重用符合情境的既有 template；修改後仍須實際重跑。此來源只確認崩潰與修正動作，沒有重跑成功結果。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
