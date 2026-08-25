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

# 參數化 K policy 修正

不可把 set4 的 K6/K8 寫成固定 K。應以 `r = K_output / K_hint` 表示，並沿用現有
`N/K < 8` 定義，分別調整 sparse 與 non-sparse high-K：

```text
K < 16:                 r_low = 1
K >= 16 and N/K < 8:    r = r_sparse
K >= 16 and N/K >= 8:   r = r_dense
K_initial = round(r * K_hint)
```

比例必須在 causal-core post-processing 後，以未加權 macro BA 直接選擇；不設 TPR、
TNR、worst-case 或「維持 exact-K」的額外 gate。現有 GMM pseudo-BA 不參與這個
選擇，因為它在 set4/formal 都錯誤偏好最小候選 K。

目前 strict 可跑 evidence 的初值：

- sparse PRgen：`r_sparse=1/4`。N48/K16 為 K4、BA `0.675000`；N57/K19
  為 K5、BA `0.649123`，均高於其他測試比例。
- non-sparse N300/K16：set4 與舊 formal 的 causal-core macro BA 在
  `r_dense=3/8` 最高，分別為 `0.841080`、`0.843183`，macro `0.842132`；
  `r=1/2` macro `0.840800`，`r=1` macro `0.797693`。

這些是初值，不是已 promotion 常數。完整 20-set 當前 strict 重跑被
`benchmark_set_10` 的 `ValueError: first Mismatch block has no parseable Ibex/Spike pair`
擋住；不得改成 non-strict 或跳過該 profile 來宣稱完整最佳比例。
