---
title: "座標零值正規化要有獨立測試"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: cfdbe7bbe9763aa73207889e2ba49728fa0229c20456a733a6d74d43e348c042
---

# 座標零值正規化要有獨立測試

助手回報向量 negate 與旋轉產出負零，部分相等性比較能區分正負零；先前 property test 使用同一組運算造期望值，兩側一起產生負零而通過。

若座標契約要求 canonical 正零，在建立向量時一致正規化，並用獨立的零向量與不同旋轉、反轉組合檢查。不要刪除零案例，或只把 expected 改成跟錯誤實作相同。

來源包含後續修正意圖但沒有最終通過輸出；歷史 RTL 反向注入與這個數值表示問題無關，不併入本筆。
