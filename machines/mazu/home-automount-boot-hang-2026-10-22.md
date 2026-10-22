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

# 結果

- 2026-10-22 套用 26.04.1／kernel `7.0.0-38` 後第一次重開，Mazu 卡在 Plymouth，LAN 上 ARP 不回應。不是 kernel 問題。
- 原因：2026-09-15 home 切換把 `/home` 改成 fstab `x-systemd.automount`，當時「沒有重開驗證」。開機 8.8 s 時 `systemd-oomd`（ProtectHome）建立 namespace 觸發 `home.automount`，NFS 需要網路但網路尚未起來；`systemd-resolved`、`systemd-timesyncd`、`systemd-oomd` 每 90 s timeout 重啟無限循環，NetworkManager 等不到。journal 證據：`home.automount: Got automount request for /home, triggered by 591 ((systemd-oomd))`。
- 修正：`/etc/fstab` 的 `/home` 行移除 `,x-systemd.automount`，保留 `_netdev,nofail`（與其他四台的一般 NFS mount 相同）。備份 `/etc/fstab.bak-20261022`。修正後開機 36 s、failed units 空、`/home` nfs4、`/apps`、HAPI hub/runner/tunnel、NVIDIA 580.178.04 正常。
- 禁止再把 `/home` 改回 x-systemd.automount。`/apps` 的 automount 五台皆有且可正常開機，未變更。

# 救援方式（root 密碼已鎖、home 在 NFS）

- GRUB `e` 在 linux 行加 `systemd.debug_shell systemd.mask=home.automount`：開機可完成，`Ctrl+Alt+F9` 是免密碼 root shell。此時 NIS 帳號（含 swear02）SSH key 登入會失敗，因為 authorized_keys 在 NFS home。
- 每台都有本機 sudo 帳號，名稱同 hostname，home 在本機磁碟（例如 `/mnt/md0/home/<host>`），可在 `/home` 不可用時登入。其密碼為弱密碼且 sshd 未關 PasswordAuthentication，屬待處理的資安風險。

# 相關

- [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]]
- NIS：Mazu、Valkyrie、Cthulhu 的 `yp.conf` 以 `zeus.ee.ntu.edu.tw`（公網 IP）連 Zeus ypserv，至少一週持續大量 `yp_bind_client_create_v3: RPC: Unable to receive`；與本事件無關，尚未修。

# 2026-10-22 後續

- Cthulhu、Zeus 重開完成：26.04.1、kernel `7.0.0-38`、NVIDIA 580.178.04 正常。Valkyrie 已 full-upgrade 但仍跑 7.0.0-31，因 ferayer（qsyn，21 天）與 chenying（GPU python，7 天）長工作未重開。重開前已確認三台 `/home` 為一般 NFS mount、唯一 automount 是 `/apps`，且 fstab 9/14 修改後已於 9/15 開機驗證過。
- NIS：四台 `/etc/yp.conf` 改為 `domain ee.ntu.edu.tw server 192.168.1.201`（Zeus LAN IP；備份 `/etc/yp.conf.bak-20261022`）。Zeus `ypserv.securenets` 已含 192.168.1.201–207。之後 ssh、pipewire、wireplumber、(systemd) 的 `yp_bind_client_create_v3: RPC: Unable to receive` 消失；剩 `systemd-logind`，因 Ubuntu 預設 `IPAddressDeny=any`＋`RestrictAddressFamilies=AF_UNIX AF_NETLINK`，無法走 IP 查 NIS，會 fallback 到 SSSD，屬噪音。
- Cthulhu 開機時 SSSD `dvlab-nis`（proxy → libnss_nis）backend 被 watchdog 殺兩次，linger 使用者 jimmy3125、piohuang、swear01 的 `user@` timeout（上一個 boot 也有 12 次 user@ timeout，非新問題）。`systemctl start user@<uid>` 後恢復，swear01 的 hapi-runner、deepseek-gateway 與 jimmy3125 的 hermes-gateway 重新運行。重開後要檢查 linger 使用者的 `user@`。
- 四台都同時有 `/etc/logrotate.d/cloud-init` 與 `cloud-init-base`，logrotate 自 7 月起每日因 duplicate log entry 失敗；未處理。
- 主機名同名本機 sudo 帳號的弱密碼為實驗室慣例，使用者明確要求保留，不要再建議更改。
