---
title: ICCAD2026B causal signature 與 soft-K 的交互作用
scope: projects/ICCAD2026B
project: ICCAD2026B
tags: [clustering, causal-signature, soft-k, ibex]
status: verified
created: 2026-08-25
updated: 2026-08-26
---

- **Causal signature lowers BA while the chosen K stays unchanged.** Reward inside the full distance matrix changes early average-linkage merges. Preserve the original hierarchy and K policy, then after cut split out a causal core when at least two cases share a signature and `hits/opportunities > 0.5`. Treat missing signature as neutral; preserve clustering for cases outside this core. All six adaptive PRgen profiles improved under this scoped alternative.
- **A good soft cut on set4 tempts removal of the dense exact gate.** N300/K16 set4 exact+core BA 0.7436, cut K6+core 0.8411; old formal holdout exact+core 0.8518 versus soft K7+core 0.7809. GMM pseudo-BA leaned to smallest K, elbow picked K3 for both, and silhouette missed the set4 sweet spot. Evaluate cut selection separately from signature integration and keep family-blocked holdouts.
- **Copying a fixed K or treating sparse policy as a smooth curve.** Calibrate `K_output / K_hint` separately by `N/K_hint < 8`; old production jumps from 1/3 to exact near that boundary. K<16 stayed exact in offline experiments. The support=27 candidate improved BA +0.0090 on 14/20 strict runnable profiles, but had worse high-K family-blocked BA than fixed sparse 1/4 (0.7132 vs 0.7180), so it was not promoted. Keep all excluded scanner failures and audit real labels.

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ICCAD2026B/causal-signature-soft-k.md)
