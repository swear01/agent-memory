---
title: "Catalog 收緊後先修正不合契約的舊 fixture"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 96d13a56d7f09d9e2a3046078ebf64a3e5103a82cc74a18b82bb5071148d88a0
---

# Catalog 收緊後先修正不合契約的舊 fixture

歷史稽核指出 strict catalog 正確拒絕舊 recipe fixture 的預設 typeId t，導致 targeted tests 失敗。

核對失敗輸入是否符合新的正式契約；若 fixture 已過時，改成有效 catalog 項目，不為讓舊測試變綠而恢復已移除的 fallback。保留非法 typeId 的拒絕測試。

來源提出修改方向，沒有修後重跑。經濟平衡的 even-split 勝負屬另一個假設，本筆不採用其全面勝出的承諾。
