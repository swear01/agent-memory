---
title: Mazu 上做計時實驗要綁 E-core（taskset -c 16-31），不綁會差到近 2 倍
scope: machines/mazu
status: active
created: 2026-10-24
updated: 2026-10-24
tags: [mazu, benchmarking, timing, taskset, e-core, thermal, ric3]
---

# Mazu 計時實驗：綁 16 個 E-core、一次只跑一個

Mazu 是 i9-14900K：CPU 0–15 是 8 個 P-core（含 hyperthreading，約 5.7 GHz），CPU 16–31 是 16 個 E-core（約 4.4 GHz，每 4 顆共用一個 L2），governor 為 `powersave`。

2026-10-24 用 rIC3 master portfolio（16 個 worker）在 PicoRV32 跑 9 題 × 2 個模型 × 3 次，一次只跑一個：

- **不綁核**：同一個模型的最慢/最快比，中位數 1.62，最大 1.95。前一分鐘 package 會到 100 °C（crit），約 5.2 GHz，之後掉到 4.1/3.3 GHz，throttle 計數一路上升。worker 會被分散到 P-core 與 E-core 上，每次分配不同。這不是外部負載造成的，跑之前 load 很低。
- **`taskset -c 16-31`**：最慢/最快比，中位數 1.02，最大 1.07。E-core 穩定在 4.4 GHz、75–78 °C，throttle 計數不再增加。fork 出來的 worker 會繼承 affinity。

做法：`taskset -c 16-31 <runner>`、parallel 1，並在輸出中記錄 `os.sched_getaffinity(0)`。代價是單次時間會變長（例如 ADD B0 不綁核的中位數約 89 s，綁核後 122 s），不同 pinning 方式量到的數字不能混著比較。量測紀錄見 [[projects/rtl-relate/neuroabs-decision-experiment-2026-10-22]]。
