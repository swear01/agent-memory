---
title: "極小 benchmark 組併入既有 runner"
scope: projects/cpachecker
status: active
updated: 2026-09-08
evidence_digest: 7b2315c99d640e168fb399873425fbd59d39f7036412257996237312c6d114fe
---

# 極小 benchmark 組併入既有 runner

## Problem

助手為一題 solved 與另一組較大的 unknown benchmark 分別建立背景 session，使用者明確要求放在同一個 session。

## Durable rule

同一輪 benchmark 中，只有一題的小組沿用既有 session／runner，仍分別保留輸出與模型介入狀態；不要只因分組就另開一個持續背景工作。

## Boundary

適用於可共用生命週期的相關 benchmark 組；不同隔離需求不由這次糾正推定。歷史助手承諾下次改進，來源未證明已重跑。

## Verification

控制者核對保留的歷史訊息及同來源鄰近訊息，確認上述使用者糾正。未重新執行歷史測試，不把助手陳述或修復計畫當成已驗證的結果。
