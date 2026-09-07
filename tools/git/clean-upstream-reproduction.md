---
title: "先在乾淨上游確認缺陷再提出修正"
scope: tools/git
status: active
updated: 2026-09-08
evidence_digest: 5aad516a9b53fb7d8f526edcbe301c672f2c9a19b9c9ebb6fafbfbca70345f89
---

# 先在乾淨上游確認缺陷再提出修正

## Problem

使用者指出，工作樹版本表現良好並不證明上游需要修復；歷史記錄另指出，本地捲動卡頓混入了前一個 session 留下的未提交 guard。

## Durable rule

以獨立、乾淨且版本明確的上游工作樹對照重現，分開記錄上游缺陷與本地未提交修改造成的行為。保留既有髒工作樹，不為了取得乾淨基線而丟棄他人修改。

## Boundary

適用於共享 checkout 或多個 agent 曾修改的專案。歷史助手對特定 guard 的原因分析屬來源陳述，不能代替新的瀏覽器重現。

## Verification

控制者核對保留的歷史訊息及同來源鄰近訊息，確認上述使用者糾正。未重新執行歷史測試，不把助手陳述或修復計畫當成已驗證的結果。
