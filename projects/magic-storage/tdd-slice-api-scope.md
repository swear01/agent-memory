---
title: "第一個 TDD slice 不能變成未經同意的產品限制"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 3086821c3f5fd55fd05dec78f9f7faab8d0c986261856df7b4fa98b7eb30977d
---

# 第一個 TDD slice 不能變成未經同意的產品限制

歷史助手承認把單輸入到單輸出的第一個測試切片誤當公開 API 邊界；使用者要求支援 deterministic 行為並明確納入 fluid／chemical，排除 external-machine send-and-wait。

先保留需求允許的能力範圍，再按可驗證切片實作，不因起步測試小就縮小產品。來源的廣義 transaction API 與 issue 都尚未完成；不能把較早的純 item 提案寫成使用者最後決定。
