---
title: "Query 參數不存在時先判存在，再轉數字"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 6d96b4d35223872d9f6ab4880efcfeac1d3708f3570a58524d2d87e066023f13
---

# Query 參數不存在時先判存在，再轉數字

Playwright 期待 seed14，畫面卻顯示 seed0；歷史助手定位到 Number(q.get("seed")) 將缺席值 null 轉成0，而整數檢查接受了0。

先區分參數不存在、空字串與提供的值，再轉型並驗證合法範圍。避免預設值因 coercion 被悄悄覆蓋；其他參數即使剛好被不同範圍檢查擋住，也應遵守同一存在性契約。來源只到修正提案，沒有測試重跑結果。
