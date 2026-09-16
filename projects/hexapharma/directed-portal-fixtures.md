---
title: "Portal fixture 要符合有向配對契約"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 37d76059b986c3ba39edd245ab24e4d1876059e80cc598ba28a24f66ae15fe81
---

# Portal fixture 要符合有向配對契約

歷史助手回報修正連鎖與共享出口 fixture，使 Portal B 唯一、不再啟動，拒絕自傳送及 B 同時作為 A；未揭露兩端時亦不顯示配對方向。

先確認遊戲的有向配對規則，再讓 validation、生成器、fixture 與呈現一致。錯誤雙向假設可能使測試本身接受非法資料。來源僅有修改與測試自述，沒有可重跑 diff；線性反查在當次地圖可接受也不是普遍效能保證。
