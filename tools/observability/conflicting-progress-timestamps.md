---
title: "矛盾時間戳不能直接換算停滯時間"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: cd8630cf64af117876293078248a6b46815e2fcd4ff34c48c33dffa3b3506027
---

# 矛盾時間戳不能直接換算停滯時間

歷史狀態輸出列出的下一次 call 時間竟晚於目前時間；助手又發現檔案 mtime 與該日誌時間不一致，轉查 sidecar。

計算等待或停滯前，確認時間來源、時區與 run 身分可比。mtime 只是交叉線索，不自動取代日誌時間；未釐清時報告時間不確定。來源停在查核中，沒有確定延遲或時鐘根因。
