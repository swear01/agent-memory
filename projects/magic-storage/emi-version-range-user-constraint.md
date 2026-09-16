---
title: "EMI internal bridge 的版本綁定不能越過使用者相容性要求"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 0f9d19285798ada655a7b5cc287fde93a2f206523b1dee5854e45c444f336487
---

# EMI internal bridge 的版本綁定不能越過使用者相容性要求

助手研究方案為讀 Tree internal fields 提議 client exact-version，使用者明確拒絕並要求接受後續 EMI 版本；同一來源又修正先前認為 Tree 完全忽略 handler inventory 的說法。

優先核對公開 API 與可維護的相容邊界，將 UI 資源顯示和 server 權威 planner／交易分開。不要把當時 inspected tag 的存在當採用 exact pin 的授權，也不能直接宣稱未知未來版本必相容；來源沒有後續設計、實作或版本範圍驗證。
