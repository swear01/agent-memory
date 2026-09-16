---
title: "資源圖示必須能讀到個別 key"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 456c8e5e7edd4c37692590510730975fbd67794794bffa4f68fef23569a85ebd
---

# 資源圖示必須能讀到個別 key

歷史唯讀分析指出，Gas 共用 Brewing Stand 圖示，原因是 StorageResourceKind 的 Supplier<ItemStack> 只為每個 kind 提供一個代表物，呈現介面看不到個別 chemical key。來源把這個問題與 chemical key 損壞區分開來。

如果同一 kind 內的資源必須呈現不同圖示，先檢查呈現介面是否收到個別 key，以及 client renderer 如何解析它。只改顯示名稱可以修正文字，不能證明共用圖示已改變。名稱與圖示應分別驗證。

證據是歷史靜態分析的保留片段，沒有本次重跑的 build、GameTest 或 GUI；client chemical sprite renderer 是當時提出的方向，並非已完成修復。
