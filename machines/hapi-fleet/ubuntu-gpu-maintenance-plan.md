---
title: 實驗室 Ubuntu、NVIDIA 與 CUDA 分階段維護計畫
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-22
updated: 2026-11-19
tags:
  - ubuntu
  - nvidia
  - cuda
  - bios
  - maintenance
---

# Ubuntu GPU 主機維護：driver mismatch、開機和 NFS

- `nvidia-smi` 的 CUDA 欄是 driver 能力。另看 `nvcc --version`、實際 Toolkit symlink 和專案 smoke。Zeus GTX 1080 Ti 是 Pascal `sm_61`，留 NVIDIA 580 和 CUDA 12.9。595 的 modalias 缺 PCI ID `10de:1B06`；Ada 主機用 13.3 Update 1 與其相容分支。Zeus 的 CUDA 12.9.1 是 runfile 安裝，Ubuntu 26.04 不在該 Toolkit 當時驗證矩陣，使用者接受這個例外。不要順便更新 Zeus BIOS。見 [Zeus Toolkit](../zeus/cuda-toolkit-system-install.md)。
- 若 `Driver/library version mismatch`，比對 loaded `/proc/driver/nvidia/version`、當前及預定 kernel 的 on-disk modinfo、userspace NVML/libcuda symlink、APT/DKMS 和 journal。未重開的舊模組可能與新 library 不合。Mazu 使用 Ubuntu 預編譯 `linux-modules-nvidia-*`，空 DKMS 清單不證明沒裝。Firmware package 版本也不證明 active driver。
- 先下載，核對維護窗口和工作負載，再靠近核准重開時安裝，縮短新 library 與舊模組並存時間。逐台保留已知可開 kernel 和本機管理入口。重開後驗 boot ID、loaded/on-disk driver、`nvidia-smi`、GPU smoke、SSH／NIS／NFS／HAPI／Docker。遠端無回覆不等於主機停機。Athena 復發時停止擴大變更，見 [近期 freeze](../athena/recurring-freeze-2026-11.md)。
- 該五台依使用者決定停 APT 自動更新。查 `APT::Periodic::Update-Package-Lists`、`Unattended-Upgrade` 均 0，兩個 apt-daily timers disabled/inactive。2026-11-18 使用者解除四台 Snap refresh hold；Zeus 沒有 Snap。APT 與 Snap 狀態分開核對。設定不適用 Oracle/Mac/Windows/NAS，也不阻止人工更新。原檔在各台 `/var/backups/disable-auto-updates-20260915-*`。

## 開機被 NFS 卡住

- 若 `snapd.apparmor.service` Starting 不結束，查 journal 是否觸發 `apps.automount`，及 `/etc/environment` 的 PATH 是否含 `/apps/bin`。NFS 等 network-online，Snap 在 sysinit 前，網路在 sysinit 後，形成等待環。早期 service PATH 只用本機，登入由 `/etc/profile.d/zz-dvlab-apps-path.sh` 加共享入口，batch SSH 明確 source 選版。Recovery 若暫時 mask Snap AppArmor，修後 unmask 並查 journal。保留實際 AppArmor 保護，Snap refresh hold 不等同停用保護。
- Athena/Mazu 曾以正常 reboot 驗證 PATH 修正。五台已修；Valkyrie/Cthulhu 當時在線部署且查 service，未同批重開；Zeus 無 Snap，網路查 systemd-networkd。相同 Docker tag 與不同 image ID 都不足以判斷內容，另看 RootFS layers 與 entrypoint。
- Mazu 的 fstab `/home` 保留原 export，`autofs.service` 不再管理。2026-10-22 已依 [開機卡住證據](../mazu/home-automount-boot-hang-2026-10-22.md) 移除 `/home` 的 `x-systemd.automount`，保留 `_netdev,nofail`；`/apps` 的 automount 維持。各機掛載選項仍不同。切換先以暫存 mount 對 device/inode/UID/GID，備份 fstab/auto maps，確認使用者 work 的停止授權，從本機 disk 執行正常卸載。Lazy/force unmount 可能遺留失效 cwd。用 `findmnt -rn -t nfs4 -T <home-user> -o SOURCE`，避免同時讀到 autofs 層而誤判。失敗還原原設定；成功由使用者實測 write/read/fsync，確認 NIS、SSH、GPU、Docker、Hub/Tunnel/Runner。
- Recovery 的 `sulogin` 密碼／Enter 提示可由 root 密碼狀態造成。先用 `passwd -S root` 核對，不輸出 hash。Valkyrie 曾有 root 密碼，後依授權鎖定並驗 sudo／隔離 PTY。提示不同無法單獨證明 boot mode 不同。另見 [升級復原與 NIS](ubuntu-upgrade-recovery-runbook.md)。

- 2026-11-19 Ada 四主機部署 R595 並採 Canonical 預編核心模組；換系列前用 `apt-cache show nvidia-driver-<ver>-server` 的 Modaliases 對照 `lspci -nn -d 10de:`。不要只排固定 reboot 時間，先確認 dpkg 無 pending 且已裝對應 kernel 模組。Cthulhu/Valkyrie 曾在排程重開時仍跑 postinst，修復見 [安裝收尾](firmware-notifier-snap-maintenance-20261118.md#2026-11-19-nvidia-安裝收尾)。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/ubuntu-gpu-maintenance-plan.md)
