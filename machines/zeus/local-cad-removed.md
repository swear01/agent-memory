---
title: Zeus 本機 /cad 已刪與 fuser -m 誤判
scope: machines/zeus
machine: zeus
status: active
created: 2026-09-14
updated: 2026-09-14
tags:
  - cad
  - fuser
  - genus
  - conformal
  - apps
---

# 結果

- 2026-09-14 在 Zeus（SSH `su_zeus`）刪掉本機 `/cad`（約 7.2G）與 `/usr/cad` → `/cad` symlink。`test ! -e /cad && test ! -e /usr/cad` 印出 `DELETED`。沒有重建 `/usr/cad`。
- 樹內是舊 Cadence Genus／Conformal 與 `eda.bashrc`，外加 `synopsys/verdi` 指到學生家目錄的 symlink。沒有把 Genus／Conformal 遷到 `/apps`。`rm -rf /cad` 只刪 symlink，不刪學生樹。
- `libpng12-0` 仍安裝，另案再決定。NFS `/apps` 仍是 `192.168.1.200:/volume1/apps` nfs4。Login `vcs`／`verdi` 仍是 `/apps/bin/...`。

# `fuser -m` 陷阱

- `/cad` 不是獨立掛載，而是 `/`（`/dev/sda2` ext4）上的目錄。
- `fuser -vm /cad` 會列出整顆 root 檔案系統的使用者（kernel `mount /`、systemd、所有 kernel thread、當下 SSH），看起來永遠「非空」。
- 真正要看有沒有人握著這個目錄：`fuser -v /cad /usr/cad`（不要 `-m`），再加上 `lsof` 路徑符合 `^/cad` 或 `^/usr/cad`。刪除當下這兩項都是空的。
- 不要把 root-fs 雜訊當成 EDA 使用中，也不要殺那些行程。
