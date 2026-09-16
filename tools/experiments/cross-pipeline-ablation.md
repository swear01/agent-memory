---
title: "來源管線的分數改善不能直接轉移到另一條管線"
scope: tools/experiments
status: active
updated: 2026-09-16
evidence_digest: 879f216b4884779ac3c46321df7b9047afad2b088ac48a2ddcf41eaf16677ecd
---

# 來源管線的分數改善不能直接轉移到另一條管線

歷史對照表回報 vectorized-only 與目標 baseline 分數相同；移植的 failure_mode_diff 卻使一組分數由 1.000000 降至 0.722222，另一個距離項沒有帶來來源管線聲稱的提升。

先保持 baseline 等價，再把各個新特徵或距離項分開做目標管線對照。來源分數、正規化方式與特徵集合不同時，不把來源提升當本地效益。

數字是保存的助手報告，未獨立重跑；不保留未核對的通用四倍加速承諾，也不將單組退化推成所有資料都無效。
