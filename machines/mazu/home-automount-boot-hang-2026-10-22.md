---
title: Mazu /home 的 x-systemd.automount 造成開機卡死（2026-10-22 已修）
scope: machines/mazu
machine: mazu
status: active
confidence: high
created: 2026-10-22
updated: 2026-10-22
tags:
  - boot
  - nfs
  - automount
  - protecthome
---

# Mazu 開機卡 Plymouth：NFS /home automount

- 更新後重開卡在 Plymouth 或 LAN ARP 不回，不要先認定是 kernel。若 journal 有 `home.automount: Got automount request for /home, triggered by ... systemd-oomd`，且 resolved／timesyncd／oomd 各等待 90 秒，先查 `ProtectHome` 建立 namespace 提早觸發需網路的 NFS，導致 NetworkManager 起不來。
- 2026-10-22 修正 `/etc/fstab` 的 `/home` 行，移除 `x-systemd.automount`，保留 `_netdev,nofail` 與原 export。備份為 `/etc/fstab.bak-20261022`，不直接還原會卡開機的行。與有既有用途且已驗的 `/apps` automount 分開。重開後核對 `/home` 的 nfs4 mount 和代表檔，以及 failed units、SSH／NIS、HAPI Hub/Runner/Tunnel 和 GPU。
- NFS home 未掛時，中央 SSH key 驗證不能依賴該 authorized_keys。保留可到場的本機管理入口，由授權維護者使用私有 DVLab MIS 的救援登入及 GRUB／recovery，先讓 `home.automount` 不介入本次開機，再修 fstab。不要在共享記憶記錄帳號的認證值、推算方式或臨時救援資料。
- 若開機後 linger 使用者服務沒回來，查 `user@<uid>` 狀態、SSSD bind 和各自 gateway/Runner，先驗身份和原因再恢復對應 unit。Zeus NIS 路徑、已更新的 SSSD 與密碼驗證以 [NIS 復原 runbook](../hapi-fleet/ubuntu-upgrade-recovery-runbook.md) 為準。Logrotate duplicate 另看 [degraded units](../hapi-fleet/ubuntu-upgrade-recovery-runbook.md#ubuntu-2604-重開後的常見-failed-units)，不混為 NFS 卡開機。
