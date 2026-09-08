---
title: "BFS 深度邊界須包含最後一層狀態"
scope: "projects/pono-llm"
status: active
updated: 2026-09-08
evidence_digest: a2b6597e851121b440e507415af694ea0dcfdf53bb449017fba699c0fec9b2f0
---

# BFS 深度邊界須包含最後一層狀態

## Problem

hot_refs_near_bad 只在迴圈處理 frontier 時收集 state；恰在最後一跳加入 next_frontier 的 state 沒有被處理，造成非空依賴圖回傳空集合。

## Durable rule

先定義 depth 是否包含邊界，再測試恰好位於邊界的節點。若契約是包含 depth 跳內的 state，就必須收集最後 frontier，並避免因此多展開一跳。

## Boundary

本次用歷史函式與附帶圖重現 depth=4 漏掉三個 hop4 state，而 depth=5 找到它們。這證明舊函式的邊界差異，未重新執行正式修復。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
