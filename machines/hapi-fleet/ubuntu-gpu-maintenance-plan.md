---
title: 實驗室 Ubuntu、NVIDIA 與 CUDA 分階段維護計畫
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-22
updated: 2026-08-24
tags:
  - ubuntu
  - nvidia
  - cuda
  - bios
  - maintenance
---

# 已確認決策

- Zeus 使用老舊 Supermicro `X10DAI`，BIOS `2.0`（2016）；使用者明確決定不更新 BIOS。
- 五台實驗室主機已完成 Ubuntu `26.04 LTS` 升級，kernel 統一為 `7.0.0-30-generic`；NVIDIA driver 統一為 `580.173.02`。
- Athena 在 BIOS `1836` 後出現的 hard freeze 原因尚未完全確認；若再次 freeze，立即停止後續 fleet 變更並重新調查。
- Zeus 的 GTX 1080 Ti 是 Pascal `sm_61`，必須保留 CUDA `12.9`；CUDA 13.x 不可套用到 Zeus。
- Mazu、Cthulhu、Athena、Valkyrie 統一使用 CUDA Toolkit `13.3 Update 1`；Zeus 單獨維持 CUDA `12.9` 相容線。
- Zeus 的 CUDA `12.9` Toolkit 尚未安裝；目前只有 R580 driver。不可把 `nvidia-smi` 顯示的 CUDA capability 當成已安裝 Toolkit。

# CUDA 版本判斷（2026-08-22）

- NVIDIA 官方目前最新 CUDA Toolkit 是 `13.3 Update 1`；CUDA 13.x 支援 Turing 及更新架構，Pascal 的最後 Toolkit 支援線是 CUDA 12.x。
- Ada 主機已依後續使用者決策安裝 CUDA `13.3 Update 1`；四台各有 37 個 CUDA 套件，`cuda-toolkit-13-3=13.3.1-1`，沒有其他 CUDA Toolkit 分支殘留。
- Zeus 已升到 Ubuntu `26.04`，使用者接受 CUDA `12.9` 在該 OS 上未列官方驗證矩陣的風險；安裝計畫仍需使用獨立的 Toolkit 來源，不加入 CUDA 13.x repository。
- Athena 目前不是 live freeze；freeze recurrence 仍是立即停止條件。

# Athena freeze 現況複查（2026-08-22 17:17）

- Athena 目前不是 live freeze 狀態：SSH 可用、目前 boot 自 `16:13:19` 已運行約 `1:03`，`nvidia-smi` 正常，RTX 4090 約 `37°C`、閒置。
- 本次 current boot 與上一個異常 boot 都沒有新的 `NVRM Xid`、GPU 掉線、PCIe AER、MCE、watchdog、OOM、NVMe I/O 或 ext4 failure 證據。
- 但是相同的 GPU ACPI `PEG1.PEGP._DSM` / `AE_ALREADY_EXISTS` 錯誤仍在本次 boot 重現；上一個 boot 仍符合 hard freeze/manual reset 的歷史證據。
- 判定：Athena 現在沒有正在 freeze，freeze 根因仍未證明解決；依使用者風險接受決定，可納入 R580 + CUDA 12.9 維護，但再次 freeze 就停止後續變更。HAPI hub/tunnel/stray-runner 錯誤另行處理，不作為 freeze 根因。

# NVIDIA driver 與 CUDA 版本現況（2026-08-24）

- CUDA `12.9 Update 1` 的 Linux 完整 Toolkit driver 最低為 `R575.57.08`；五台目前均為 Ubuntu 套件的 `R580.173.02`，不把 driver 版本誤稱為 CUDA 版本。
- `mazu`、`cthulhu`、`athena`、`valkyrie`：CUDA Toolkit `13.3.1-1`，`/usr/local/cuda` 指向 `cuda-13.3`，沒有其他 Toolkit 分支。
- `zeus`：GTX 1080 Ti + R580.173.02；沒有 `/usr/local/cuda`、`nvcc` 或 CUDA Toolkit 套件，CUDA `12.9` 安裝仍待執行。

# 2026-08-24 統一結果

- 五台 Ubuntu main archive 統一為官方登記的 `https://mirror.twds.com.tw/ubuntu/`；Docker source 與 Ubuntu security source 亦保持一致。每台 `apt-get update` 與 `apt-get -s full-upgrade` 均通過，沒有待安裝或移除套件。
- 五台共同安裝的 1,635 個 APT 套件逐包比對後版本漂移為 0；各機因角色不同，安裝套件總集合不要求完全相同。
- 五台共同的 system administration baseline 為 `ethtool`、`openssh-server`、`lsof`、`dmsetup`、`lm-sensors`、`smartmontools`、`pciutils`、`dnsutils`、`tcpdump`、`rsync`、`sysstat`、`curl`、`wget`、`unzip`、`zip`；2026-08-24 由 `swear02` 逐台 live 查核，15 項全部存在且版本完全一致，`dpkg --audit=0`、`apt-get check` 通過。
- HAPI、Codex、Claude Code、OpenCode、Cursor Agent、agy、Node、uv 與 user-local JDK 屬於個人／HAPI runner 使用者環境，不是 system-wide 必裝基線。Zeus 唯一的 system-level AI CLI 痕跡 `/usr/local/bin/opencode` 已移除；`swear02` 仍由 `~/.local/bin/opencode` 使用相同 user-local binary。

# Valkyrie BIOS 與 CUDA 現況

- Valkyrie 的 ASUS `PRIME Z790-A WIFI` 已完成 EZ Flash 3，讀取 DMI 確認 BIOS `1836`、日期 `04/16/2026`；因此 Cthulhu、Mazu、Athena、Valkyrie 的 BIOS 維護均已完成。Zeus 的 Supermicro BIOS 依明確決策維持不變。
- Valkyrie 目前是 Ubuntu `26.04 LTS`、kernel `7.0.0-30-generic`、RTX 4090、NVIDIA driver `580.173.02`，已安裝 CUDA Toolkit `13.3.1-1`。
- `/usr/local/cuda` 指向 `cuda-13.3`；Docker daemon 與 `swear01` HAPI runner 均 active。

# CUDA 12.x → 13.3 的性能判斷

- CUDA Toolkit 不會改變 RTX 4090 的硬體時脈、SM 數量或記憶體頻寬；對已編譯的 binary、PyTorch wheel 或自帶 CUDA userspace 的 container，單純在 host 安裝較新 Toolkit 通常不會自動變快。
- 若 workload 會用新 Toolkit 重新編譯，或實際載入新版 cuBLAS/cuDNN 等 userspace library，性能可能改變，可能變快也可能退步；不能宣稱「完全沒有差異」。
- CUDA 13.3 Update 1 對 Ada/compute capability `8.9` 不是過新，但官方 release notes 明列的 cuBLAS 性能提升主要集中在 Hopper/Blackwell，沒有 RTX 40 的普遍性能提升保證。
- 四台 Ada 主機目前以 R580 的 CUDA minor compatibility 執行 CUDA 13.3；不可把這個組合誤稱為 NVIDIA 的完整 R610/CUDA 13.3 基線。
- CUDA `12.9` freeze 只適用 Zeus 的 host Toolkit、runtime 與 workload image；四台 Ada 主機維持已驗證的 CUDA `13.3.1-1`。未來變更 driver/Toolkit 時仍需用相同 workload 做 A/B benchmark。

# 後續工作

- Zeus CUDA `12.9` 是剩餘的 GPU 維護項目；安裝前重新確認 1080 Ti 無 workload，安裝後驗證 `nvcc`、sm_61 編譯與實際執行。
- 不改變既有 HAPI runner ownership；active SQLite/WAL state 留在 host-local `/var/tmp`，不可同步回 NFS。
- Release-upgrade recovery and local-vs-centralized authentication evidence are maintained in `ubuntu-upgrade-recovery-runbook.md`。
