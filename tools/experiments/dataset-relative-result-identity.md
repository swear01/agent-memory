---
title: "跨目錄同名案例不能用 basename 當結果身份"
scope: tools/experiments
status: active
updated: 2026-09-16
evidence_digest: a9bb5585dd89a4067b80bf302f2a0df42f64f9e50cd4af801ba75c7234b0de1c
---

# 跨目錄同名案例不能用 basename 當結果身份

歷史結果更正明說案例 key 碰撞造成少算，改用包含路徑的 key 後重新計數。相鄰來源含不同 property 與 run 的數字，不能混成同一總表。

結果帳冊使用在固定 dataset 中唯一且可移植的身份，例如 dataset-relative path 加資料集版本；合併前檢查唯一性與輸入輸出對應。可由完整既存結果重算時，不必重新跑實驗。

來源支持碰撞與重計數的報告，本筆不採用相鄰互不一致的成功數、PAR-2 或零退化宣稱。
