---
title: ICCAD2026B causal signature 與 soft-K 的交互作用
project: ICCAD2026B
tags: [clustering, causal-signature, soft-k, ibex]
status: verified
created: 2026-08-25
updated: 2026-08-26
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

# 2026-08-26 sparsity-only ratio 曲線稽核

現行 production 不是平滑 scaling curve，而是 `K >= 16` 時以 `N/K < 8`
為界，在 `r=1/3` 與 `r=1` 間跳轉。這會讓 K16 在 N127 時輸出 K5、N128
時直接輸出 K16；K32 也會由 N255 的 K11 跳到 N256 的 K32。

目前 name-free distance 上，對 Vichuang 四個 strict 可跑 sparse profiles
重新建立同一棵 average-linkage hierarchy，並把搜尋範圍由 K2 掃到 K_hint，沒有
使用舊 `K/4` floor：

- 共用 ratio 的最佳 rounding plateau 是 `0.2345..0.2655`，輸出 K 為
  `4/4/8/4`。`r=0.21875` 輸出 `4/4/7/4`，macro BA 只低 `0.000266`；
  `r=0.1875` 輸出 `3/3/6/3`，macro BA 由 `0.646093` 降到 `0.608506`。
- 下於 `1/4` 可成為單一 profile 的 oracle：set4_1 最佳 K2 (`r=0.125`)，
  set8_1 的次佳 K3 (`r=0.1875`)。但 set8_1 的全域最佳是 K11
  (`r=0.6875`)。
- set4_1 與 set8_1 的 `N/K` 完全同為 `6.8125`，oracle ratio 卻是
  `0.125` 與 `0.6875`。因此任何只看 sparsity 的 deterministic `r=f(N/K)`
  都不可能同時選到兩者最佳；它最多是跨 profile 的平均政策。

歷史 beta soft-K 也有同方向反例：相近約七 cases-per-hint 的 K64 profiles 曾
分別選 K31 與 K59。可取消研究搜尋的 `1/4` 下限，但 production curve 必須用
family-blocked holdout 比較 fixed `1/3`、fixed `1/4` 與 GMM；不能因同一資料集的
oracle 低於 `1/4` 就 promotion。

`adaptive_sparse_k` 另是一個名稱容易混淆的 feature calibration：它只把
test/class/bin identity group scale 在 sparse regime 乘 `0.25`，不計算 K ratio；
PR166 已把這三組 identity features 預設關閉，所以目前 default distance 不存在
可學習的 identity scaling curve。

# 2026-08-26 離線 support curve

將 high-K 補償限制成一個參數，避免學 profile-specific 曲線：

```text
K_hint < 16:  target_k = K_hint
K_hint >= 16: r = min(1, (N / K_hint) / support)
              target_k = max(2, round(N / support))
```

Vichuang strict native-scanner 重跑仍只有 14/20 可用；六個失敗與前次完全相同，
沒有用 non-strict fallback 或補值。七個 high-K profiles 的全資料最佳為
`support=26`；leave-one-family-out fit 分別為 `33/24/27/26`，取中位數後的研究
候選為 `support=27`。它在四個 sparse profiles 輸出 `K4/K4/K8/K4`，兩個
N300/K16 dense profiles 輸出 K11，N1000/K32 保持 K32；所有 K8 保持 exact。

14-profile macro（profile 等權）：

- production sparse 1/3：BA/TPR/TNR `0.698691/0.622975/0.774407`
- sparse 1/4：`0.701891/0.635132/0.768649`
- support=27 curve：`0.707722/0.652507/0.762938`

curve 對 production 1/3 為 BA `+0.009031`、TPR `+0.029532`、TNR
`-0.011469`。但 high-K family-blocked CV BA 只有 `0.713228`；exact、1/3、1/4
在同列分別為 `0.714417/0.710434/0.717950`。所以這條曲線只證明同資料的潛力，
尚未勝過 fixed 1/4 的 family holdout，不可 promotion 到 production。
