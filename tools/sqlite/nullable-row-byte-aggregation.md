---
title: "可空欄位的 bytes 加總不能漏掉整列"
scope: tools/sqlite
status: active
updated: 2026-09-16
evidence_digest: 8c116cff2dc1e5ce0d6600ae8f3cfa0813f27e4042b52253ce70dbc1e881f7f5
---

# 可空欄位的 bytes 加總不能漏掉整列

助手定位 messages 的可空欄位使 LENGTH 結果為 NULL，逐列長度算式跟著成 NULL，SUM 因而略過那些列。

依統計契約在各可空欄位處定義零貢獻，再相加；測試同列混有 NULL 與非空內容。過濾掉含 NULL 的整列會繼續少算，不能當等價修正。

來源是診斷與修改意圖；相鄰 CI 成功屬另一個 SHA，不能當本修正已通過。內容長度仍不等於實體頁面配置。
