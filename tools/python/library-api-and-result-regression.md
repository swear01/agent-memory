---
title: "Library API 修通後仍須檢查結果退化"
scope: "tools/python"
status: active
updated: 2026-09-15
evidence_digest: dd89c0890960118b09dc6ac728b2ac6cc1d90f5d854ec54546dfd8177ba98e4a
---

# Library API 修通後仍須檢查結果退化

歷史執行先因 AffinityPropagation 物件沒有 converged_ 屬性而退出；修改後，同一小型案例雖產出 CSV，評分卻是 0.833333，低於要求的 1.0。後續摘要回報保留原小樣本 clustering 路徑。

採用 library 屬性前，核對已安裝版本的實際介面。排除 AttributeError 只證明程式走得更遠，不能取代原本的品質門檻；以相同資料與評分檢查新方法，退化時保留已驗證的基線或明示差異。

來源有錯誤與退化輸出；本次未重新跑 clustering，也不採用來源對小樣本最優演算法的泛化判斷。曾提議的 convergence 替代判斷不在本筆記中視為已驗證 API。
