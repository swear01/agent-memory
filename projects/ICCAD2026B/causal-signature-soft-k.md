---
title: ICCAD2026B causal signature 與 soft-K 的交互作用
project: ICCAD2026B
tags: [clustering, causal-signature, soft-k, ibex]
status: verified
created: 2026-08-25
updated: 2026-08-25
---

# 結論

跨 test causal signature 不受 exact-K 理論限制。失敗根因是把共享 signature 的
reward 直接乘進完整 distance matrix，會改寫 average-linkage 的早期 merge，進而
連鎖改變整棵 hierarchy；adaptive PRgen 即使前後都選 5 群，BA 仍可由
`0.567251` 降至 `0.524366`。

較穩定的最小方案是：保留原 clustering hierarchy 與 K policy，完成 cut 後再把
`hits/opportunities > 0.5`、至少兩 case 共享的 signature cases 抽成 causal core。
missing signature 保持 neutral，不把原 cluster 整群 must-link。

# 已驗證數字

- 六個 adaptive PRgen profiles：causal-core BA 全部提升。
  - N48/K16：`0.579167 -> 0.675000`，TPR `0.916667 -> 1.0`，TNR
    `0.241667 -> 0.350000`。
  - N57/K19：`0.567251 -> 0.649123`，TPR `0.929825 -> 1.0`，TNR
    `0.204678 -> 0.298246`。
- Vichuang set4 N300/K16：
  - exact + core：BA `0.743576`。
  - unrestricted soft K4 + core：BA `0.757156`。
  - hierarchy cut K6 + core：BA `0.841080`，TPR `0.882132`，TNR `0.800028`。
  - hierarchy cut K8 + core：BA `0.839389`，TPR `0.819820`，TNR `0.858958`。
- 舊 formal N300/K16 holdout：exact + core BA `0.851810`，soft K7 + core
  `0.780925`；因此不能全面取消 dense exact gate。

# Selector 限制

現有 two-component GMM pseudo-BA 在 set4 與 formal 都偏好最小候選 K，無法區分
兩者各自最佳 cut。Elbow 兩者都選 K3；silhouette 在 set4 選 K16、formal 選
K14，也找不到 set4 的 K6/K8 sweet spot。歷史 20-set 實驗亦記錄全面放寬後
holdout macro BA `+0.006618`，但四個 K8 profiles 退步，worst `-0.063244`。

後續應分開處理兩件事：causal core 可同時用於 exact/soft；dense high-K 是否從
exact 放寬，需另做 K-range/selector promotion，不能由 causal signature 順便決定。
