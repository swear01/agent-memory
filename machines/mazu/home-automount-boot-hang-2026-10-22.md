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
