---
title: 四台實驗室主機掛 /apps 與 Mazu 相同 login
scope: machines/hapi-fleet
status: active
created: 2026-09-14
updated: 2026-09-14
tags:
  - nfs
  - apps
  - login
  - fstab
  - vcs
---

# 結果

- 2026-09-14 在 Cthulhu、Athena、Valkyrie、Zeus 掛上同一份 Gaia share `192.168.1.200:/volume1/apps` → `/apps`，`FSTYPE=nfs4`。fstab 一行與 Mazu 相同：`rw,hard,rsize=32768,wsize=32768,_netdev,nofail,x-systemd.automount`。
- 沒有再 copy EDA 樹。四台都能 `ls /apps/{bin,cad,etc}`，`vcs.sh` 已在 share 上。
- Mazu 先前對 inode 做過 POSIX `chmod 755` 之後，這四台第一次 mount 就是 `755` / `user::rwx,group::r-x,other::r-x`，沒有 NFSv4 empty-deny。只有 `ls /apps` 失敗時才需要 `sudo chmod 755 /apps`；不要改 `nfs-home`。
- Login 檔與 Mazu 相同：`Z20-dvlab-apps.sh`（TMPDIR + source `/apps/etc/env.sh`）、`/etc/environment` 把 `/apps/bin` 放在 `/usr/local/bin` 前、`/etc/zsh/zshenv` emulate-sh source Z20。四台都有 `dvlab-env.sh` 會把 `/usr/local/bin` 插回 PATH 最前，所以也需要 `zz-dvlab-apps-path.sh`。fish 可省略。
- 不要 source 完整 `vcs.sh` / `verdi.sh` / `spyglass.sh`。不要印 license 值，不要 cat `license.sh`。不要建 `/usr/cad` symlink。
- 四台 login shell `command -v vcs` / `verdi` 都是 `/apps/bin/...`。VCS 最小 sim（`/tmp/dvlab-apps-smoke`、本機 tmpfs、`TMPDIR=/tmp`）compile+sim 結束碼 0，日誌路徑在 `/apps/cad`。測完刪 smoke 目錄。
- `/usr/local/bin/vcs` 已在後續任務從五台刪除；login 仍解析到 `/apps/bin/vcs`。同學不用手動 source：launcher 子行程會 source CIC。ICC2／3DIC／VC Formal 尚無 `/apps/bin` 入口；`/apps/bin/vcs` 尚未同時 source `verdi.sh`（FSDB）。詳見 `eda-install-inventory.md`。
- Zeus 本機 `/cad` 與 `/usr/cad` → `/cad` 已於 2026-09-14 刪除（約 7.2G Genus／Conformal 舊樹）。沒有重建 `/usr/cad`。細節見 `machines/zeus/local-cad-removed.md`。

# 注意

- 詳細 Mazu 掛載／ACL 見 `machines/mazu/apps-nfs-mount.md` 與 `machines/gaia/dsm-api-apps-share.md`。
- Login PATH 細節見 `machines/mazu/apps-login-path.md`；四台同樣適用。
