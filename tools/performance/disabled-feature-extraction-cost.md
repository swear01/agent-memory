---
title: "Feature 不進分數時仍須檢查是否繼續計算"
scope: tools/performance
status: active
updated: 2026-09-16
evidence_digest: 1e5d398ac07839559ac61032295268137b1773ab6fece72ee5cdb275a794fcf9
---

# Feature 不進分數時仍須檢查是否繼續計算

歷史助手發現停用的 sim／trace features 雖不參與分數，parser 仍計算；修改後回報 set-6 單次順序 A/B 的 parse 時間降低、BA 與 prediction SHA 不變，保留 LLM 所需模板及共用便宜欄位。

沿配置到特徵抽取的路徑停掉不需要的工作，並驗證輸出與仍啟用消費者的需求。單次共享主機量測不是跨資料集效能保證；來源相依特徵是否可作 local veto／tie-break 是後續研究，沒有因這次省解析成本而得到品質證明。
