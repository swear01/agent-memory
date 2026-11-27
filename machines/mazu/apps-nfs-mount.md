---
title: Mazu 掛載 Gaia apps 於 /apps
scope: machines/mazu
machine: mazu
status: active
created: 2026-09-11
updated: 2026-11-28
tags:
  - nfs
  - apps
  - fstab
---

# 結果

- 2026-09-11 已在 Mazu 把 Gaia share `192.168.1.200:/volume1/apps` 掛到 `/apps`，`FSTYPE=nfs4`，`vers=4.1`，`rsize=32768,wsize=32768`（與現役 home NFS 相同）。
- 現行 fstab 使用 `rw,hard,vers=4.1,rsize=32768,wsize=32768,_netdev,nofail,x-systemd.automount`。`nofail` 避免 NAS 暫時不見時卡開機；`x-systemd.automount` 讓第一次存取才掛。
- 目錄 `/apps/bin`、`/apps/etc`、`/apps/cad` 已建立，POSIX `755`。probe 檔 `/apps/.dvlab-mount-probe` 存在。沒有建立 `/usr/cad`，沒有改 `nfs-home`。
- SSH 管理帳號仍是 `swear02`（`su_mazu`），sudo NOPASSWD。

## 免重開切換 NFS 版本

2026-11-28 正式 `/apps` 和 `/home` 均從 4.0 正常卸載後以 4.1 掛回，boot ID 相同。`/apps` 保留 systemd automount。改完 fstab 做 daemon-reload，用 `systemctl cat apps.mount` 檢查產生的設定；掛載仍在使用時，`systemctl show -p Options` 可能顯示核心現役版本。重新掛載後用 `findmnt -rn -t nfs4 -M /apps -o SOURCE,OPTIONS` 排除同一路徑的 autofs 項目。

主機切換後比對各程序 mount namespace 的舊 MAJ:MIN 與 Snap 的持久 namespace。此次兩個 spec2rtl CI runner 的 bubblewrap 保留舊 /apps，在 GitHub 與各自 cgroup 確認閒置後逐一正常重啟並確認重新上線。Snap desktop integration 的持久 namespace 保留舊 /home；暫停既有使用者服務，執行套件提供的 `snap-discard-ns snapd-desktop-integration`，再恢復服務。只有在查到這些舊視圖時才需要刷新。HAPI worker 獨立的 4.0 掛載有專屬 mount unit，與主機正式掛載分開配置。

# 注意

- 剛掛上時若遇 `Permission denied`，先看 `stat`/`getfacl` 是否為 DSM 預設 ACL 造成的 mode `000`。處理方式見 `machines/gaia/dsm-api-apps-share.md`：對 `/apps` 做 `chmod 755`，不要改 `nfs-home`，也不要用 `sudo touch` 當繞過。
- 2026-09-12 複測：`/apps` 仍是 `192.168.1.200:/volume1/apps` nfs4；四個目錄 `755 root:root`、`getfacl` 為 `user::rwx,group::r-x,other::r-x`；未授權 `swear02` 可 `ls`/`cd`，`touch` 新檔 Permission denied；`/apps/.dvlab-mount-probe` 仍在。未建立 `/usr/cad`。
- 明確 `mount /apps` 後，`apps.mount` 會是 active，`apps.automount` 可能 inactive；這不代表 fstab 沒有生成 automount unit。
- 2026-09-14：同一 fstab 行掛到 Cthulhu／Athena／Valkyrie／Zeus 後立刻可 `ls`，沒有 empty-deny。後續客戶端不必預先 `chmod`。Fleet 登入與選版見 [EDA 維運](../hapi-fleet/eda-install-inventory.md)。
