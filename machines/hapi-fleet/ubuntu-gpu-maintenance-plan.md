---
title: 實驗室 Ubuntu、NVIDIA 與 CUDA 分階段維護計畫
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-22
updated: 2026-08-22
tags:
  - ubuntu
  - nvidia
  - cuda
  - bios
  - maintenance
---

# 已確認決策

- Zeus 使用老舊 Supermicro `X10DAI`，BIOS `2.0`（2016）；使用者明確決定不更新 BIOS。
- Mazu 在 Ubuntu release upgrade 前必須先修好缺失的 NVIDIA kernel module/DKMS；只安裝 userspace NVIDIA 套件不足以修復目前的 `nvidia-smi` 問題。
- Athena 在 BIOS `1836` 後出現的 hard freeze 原因尚未完全確認；完成穩定性診斷前不進行 Ubuntu release upgrade。
- Zeus 的 GTX 1080 Ti 是 Pascal `sm_61`，必須保留 CUDA `12.9`；CUDA 13.x 不可套用到 Zeus。
- Ada GPU 主機先以 CUDA `12.9` 作為穩定共同目標；只有在 Ubuntu `26.04.1`、NVIDIA driver 與實際 workload 驗證完成後，才考慮 CUDA `13.3 Update 1`，不因理論上的 performance 提升而直接升級。
- Zeus 的舊 CUDA、PPA/repository、PyPy（實際元件待盤點）與其他 legacy items 必須逐項盤點、逐項清理，不可未審查就 bulk delete。

# CUDA 版本判斷（2026-08-22）

- NVIDIA 官方目前最新 CUDA Toolkit 是 `13.3 Update 1`；CUDA 13.x 支援 Turing 及更新架構，Pascal 的最後 Toolkit 支援線是 CUDA 12.x。
- 目前不應直接把全機升到 CUDA 13.x。先以 CUDA `12.9` 作為 Ada 與 Pascal 的共同穩定目標，完成 NVIDIA driver、Ubuntu 與 workload 驗證後，再決定是否切換。
- 若最終升到 Ubuntu `26.04.1`，非 Zeus 的 Ada 主機再以 CUDA `13.3 Update 1` 為候選目標，並配合完整支援所需的 NVIDIA driver `610.43.02` 以上；R580 只能走 CUDA 13.x minor compatibility，不能當作完整 CUDA 13.3 基線。
- Zeus 必須保留 CUDA `12.9`。CUDA 12.9 官方驗證的 Ubuntu 版本目前是 22.04/24.04，未列 26.04；建議 Zeus 的 Ubuntu 停在 24.04，除非明確接受 26.04 上非官方驗證的 CUDA 12.9 組合。
- Mazu 的 NVIDIA driver 缺失與 Athena 的 freeze 都是 CUDA/Ubuntu 變更的 hard blocker。

# Valkyrie BIOS 與 CUDA 現況（2026-08-22）

- Valkyrie 的 ASUS `PRIME Z790-A WIFI` 已完成 EZ Flash 3，讀取 DMI 確認 BIOS `1836`、日期 `04/16/2026`；因此 Cthulhu、Mazu、Athena、Valkyrie 的 BIOS 維護均已完成。Zeus 的 Supermicro BIOS 依明確決策維持不變。
- Valkyrie 目前是 Ubuntu `22.04.5`、kernel `6.8.0-138-generic`、RTX 4090、NVIDIA driver `595.84`；`nvidia-smi` 顯示的是 driver capability CUDA `13.2`，不是已安裝 Toolkit。
- Valkyrie 沒有 host `nvcc` 或 `/usr/local/cuda*`，Docker daemon 目前未啟動，沒有 GPU workload；實際 `swear01` HAPI runner process 存在。`swear02` 的 stray user service 仍因 resources 失敗並保持不動。

# CUDA 12.x → 13.3 的性能判斷

- CUDA Toolkit 不會改變 RTX 4090 的硬體時脈、SM 數量或記憶體頻寬；對已編譯的 binary、PyTorch wheel 或自帶 CUDA userspace 的 container，單純在 host 安裝較新 Toolkit 通常不會自動變快。
- 若 workload 會用新 Toolkit 重新編譯，或實際載入新版 cuBLAS/cuDNN 等 userspace library，性能可能改變，可能變快也可能退步；不能宣稱「完全沒有差異」。
- CUDA 13.3 Update 1 對 Ada/compute capability `8.9` 不是過新，但官方 release notes 明列的 cuBLAS 性能提升主要集中在 Hopper/Blackwell，沒有 RTX 40 的普遍性能提升保證。
- Valkyrie 現在是 R595.84，driver capability 到 CUDA 13.2；CUDA 13.3 的完整對應 driver 是 R610.43.02 以上。若沒有 workload 需求，為了性能單獨更換到 R610/13.3 不值得增加風險。
- 若未來 Ubuntu `26.04.1` 的正式支援與 workload 相容性要求 CUDA 13.x，才在停機窗口中升級 driver/Toolkit，並用相同 workload 做 A/B benchmark。

# 升級策略

- Valkyrie 先完成現場 EZ Flash 3 與 post-flash verification。
- Ubuntu 22.04 主機依序走 `22.04 → 24.04`；Athena 從 `24.04 → 26.04.1`，其他主機待 24.04 穩定後再依序升到 `26.04.1`。
- 等待 Ubuntu 26.04.1 正式支援升級窗口；不使用 `do-release-upgrade -d`。
- 維護採一台主機、一次變更；每一步完成 workload、HAPI session、SSH、NFS、GPU、container 與 runner 驗證後才進下一步。
- 不改變既有 HAPI runner ownership；active SQLite/WAL state 留在 host-local `/var/tmp`，不可同步回 NFS。
