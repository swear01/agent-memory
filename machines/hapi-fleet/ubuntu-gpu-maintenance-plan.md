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
- 其他支援的 GPU 主機規劃使用 CUDA 13.x；確切 minor version 要等各主機的 NVIDIA driver、GPU、container 與 workload 相容性盤點後決定。
- Zeus 的舊 CUDA、PPA/repository、PyPy（實際元件待盤點）與其他 legacy items 必須逐項盤點、逐項清理，不可未審查就 bulk delete。

# 升級策略

- Valkyrie 先完成現場 EZ Flash 3 與 post-flash verification。
- Ubuntu 22.04 主機依序走 `22.04 → 24.04`；Athena 從 `24.04 → 26.04.1`，其他主機待 24.04 穩定後再依序升到 `26.04.1`。
- 等待 Ubuntu 26.04.1 正式支援升級窗口；不使用 `do-release-upgrade -d`。
- 維護採一台主機、一次變更；每一步完成 workload、HAPI session、SSH、NFS、GPU、container 與 runner 驗證後才進下一步。
- 不改變既有 HAPI runner ownership；active SQLite/WAL state 留在 host-local `/var/tmp`，不可同步回 NFS。
