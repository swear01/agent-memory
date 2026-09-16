---
title: "自動發現配方不等於能安全執行未知 family"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: f470013f082e5dfeba7b3ac7d6e8fd313a0260867cf5de0aba6666d9aaf0d699
---

# 自動發現配方不等於能安全執行未知 family

歷史方案把配方發現、公開 family API 與外部 processing pattern 混在一起；使用者選 A＋B 並明確排除外部加工模式，助手後續記錄未知且缺完整 contract 的配方繼續拒收。

以已知 family 的輸入、輸出、remainder、能源及交易契約執行，不從名稱或 ingredients 猜未知 recipe 的效果。通用 factory 是重用已知語意，不是任意模組 fallback。來源只完成文件方向，API、factory 與 compat workflow 尚未實作；早期 C 方案不應再當接受範圍。
