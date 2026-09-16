---
title: "含邊界的增量游標需要已處理事件去重"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 3c365947cb60294055d5d5792fe29e4d0b65c049aab8a97caa79cf02c328e89c
---

# 含邊界的增量游標需要已處理事件去重

歷史助手多次確認只有舊評論，將 monitor 重複通知歸因於 since 包含邊界時間，並自述已改用 comment ID 追蹤、重啟。

以穩定事件身分保存已處理狀態，讓重疊查詢或重啟不會再通知同一事件；不要假設 ID 必然單調，也須保留評論更新的契約。來源沒有後續長期觀察，且 monitor 修正不代表評論中的產品缺陷已修好。
