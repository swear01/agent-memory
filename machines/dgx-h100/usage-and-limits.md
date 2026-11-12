---
title: 重點科技研究學院 DGX H100：硬體、使用限制（每人 1 張 GPU）與 CULS 實測
scope: machines/dgx-h100
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [dgx, h100, gpu, policy, culs, benchmark]
---

連線方式見 [[machines/mazu/dgx-vpn-rootless]]（`ssh dgx`，經 mazu 的 VPN；金鑰已裝，可用 BatchMode）。

硬體與環境（2026-11-13 盤點）：8 張 H100 80GB HBM3（NVLink），2 顆 Xeon 8480CL（224 執行緒，最高 3.8 GHz），2 TB RAM。Ubuntu 22.04（glibc 2.35），driver 550.90（最高 CUDA 12.4），nvcc 12.6（/usr/local/cuda）；/usr/local/cuda-12.2 的 nvcc 在 cmake 偵測時失敗。home 是 /raid/home/<user>（/raid 剩 5.2 TB，`/` 剩 3%）。沒有 slurm；約 80 個登入 session，平常有 5 張卡被別人佔用。

**使用限制**：root 的 `enforce-gpu-policy.timer` 每 15 秒跑 /usr/local/sbin/enforce_gpu_policy.sh：每個使用者最多同時用 1 張 GPU（MAX_GPU_PER_USER=1），超過就 TERM→KILL 該使用者的 GPU process；不在 /etc/dgx_gpu_allowlist 的帳號也會被殺。2026-11-13 三卡平行測試時，有兩個 job 在 10～17 秒內被 SIGTERM（rc 143）。同一張卡跑兩個 job 不會被殺，但每個慢 1.8～1.9 倍，總吞吐量沒有增加。

**CULS 實測**（e73f3fc，resyn2rs，3 輪中位數）：單一 job 的 H100 對 4090（valkyrie）約 0.96～1.18 倍，大電路比較明顯（multiplier_12xd 1.18、hyp_9xd 1.15、log2_12xd 1.12），中小電路約 1.0；md5 和 4090 一致。原因是 host 端循序工作在 3.8 GHz 的 Xeon 上比 5.8 GHz 的 i9 慢。80 GB 的優勢只在 4090 跑不動的電路才用得到，但 div_12xd（2.34 億 AND）在兩台機器上都會在 rw replace 的 host 端 segfault。

**注意**：CULS-fast 在 #56（fad20c9）之後，用 CUDA 12.6 編會在 refactor 的 evalFactoredForm 出現 illegal memory access（在 4090 和 H100 上都會發生；用 13.3 編則正常）。DGX 只有 12.x，所以 main 在那裡跑不了，修好之前要用 e73f3fc。

**How to apply:** 在 DGX 上一次只用 1 張空卡（先確認沒有別人的 compute process，閒置卡大約佔 80～130 MiB），binary 要在 DGX 上 build（valkyrie 的 binary 需要 glibc 2.38）。它適合當作第二條獨立的驗證管線，不適合拿來平行開好幾張卡。
