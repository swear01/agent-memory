---
title: Mazu 掛載 Gaia apps 於 /apps
scope: machines/mazu
machine: mazu
status: active
created: 2026-09-11
updated: 2026-09-11
tags:
  - nfs
  - apps
  - fstab
---

# 結果

- 2026-09-11 已在 Mazu 把 Gaia share `192.168.1.200:/volume1/apps` 掛到 `/apps`，`FSTYPE=nfs4`，`vers=4.1`，`rsize=32768,wsize=32768`（與現役 home NFS 相同）。
- fstab 使用 `rw,hard,rsize=32768,wsize=32768,_netdev,nofail,x-systemd.automount`。`nofail` 避免 NAS 暫時不見時卡開機；`x-systemd.automount` 讓第一次存取才掛。
- 目錄 `/apps/bin`、`/apps/etc`、`/apps/cad` 已建立，POSIX `755`。probe 檔 `/apps/.dvlab-mount-probe` 存在。沒有建立 `/usr/cad`，沒有改 `nfs-home`。
- SSH 管理帳號仍是 `swear02`（`su_mazu`），sudo NOPASSWD。

# 注意

- 剛掛上時若遇 `Permission denied`，先看 `stat`/`getfacl` 是否為 DSM 預設 ACL 造成的 mode `000`。處理方式見 `machines/gaia/dsm-api-apps-share.md`：對 `/apps` 做 `chmod 755`，不要改 `nfs-home`。
- 明確 `mount /apps` 後，`apps.mount` 會是 active，`apps.automount` 可能 inactive；這不代表 fstab 沒有生成 automount unit。
