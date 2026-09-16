---
title: "EMI holder 不應自行加嚴 widget 座標契約"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 20615e8bc808cbd2b3e325f9e72cd735ab035ff9ade4fee0ddc6e6265135c515
---

# EMI holder 不應自行加嚴 widget 座標契約

歷史只讀調查指出自訂 holder 拒絕負座標，但 EMI bridge 可合法產生負座標及非空跨界 widget；只特判 zero-size 會漏另一條路徑。既有 static tests 通過仍沒覆蓋此條件。

遵守上游 widget 契約，保留 null／bounds 檢查，以 diagram scissor 與互動 gate 控制外圍；同時核對 render、tooltip、mouse、keyboard callers。來源只有調查與建議 GREEN，沒有 client 修復後 smoke；建議測試 recipe 不是已知 crash recipe ID，不可猜測。
