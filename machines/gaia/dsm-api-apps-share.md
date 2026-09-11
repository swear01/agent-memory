---
title: Gaia DSM Web API 與 apps NFS share
scope: machines/gaia
machine: gaia
status: active
created: 2026-09-11
updated: 2026-09-11
tags:
  - synology
  - dsm
  - nfs
  - apps
  - webapi
---

# 結果

- Gaia（Synology DS1821+，DSM 7.2.1）管理面是 DSM Web API，不是 SSH。LAN `192.168.1.200` 的 22/23 與常見替代 SSH port 皆為 connection refused；`:5000` / `:5001` 可連。
- 登入用 `SYNO.API.Auth` version 6、`account=dvlab`、`session=Core`、`format=sid`。密碼只存在 353 network setup manual，不寫進 memory。
- 瀏覽器 cookie `_SSID` 不是已登入 SID；`SYNO.Core.Desktop.Initdata` 會回 `isLogined=false`。必須重新 login。
- 2026-09-11 已建立獨立 share `apps`，File Station `real_path` 為 `/volume1/apps`。隱藏網路芳鄰、關閉資源回收筒。NFS 規則只加在這個 share：五台 LAN `.201` `.202` `.203` `.204` `.207`，`rw`、`root_squash=root`（DSM「No mapping」）、`sys`、`crossmnt=true`。客戶端路徑：`192.168.1.200:/volume1/apps`。
- 沒有改全域 NFS 服務，也沒有寫 `nfs-home` 規則。`showmount -e` 同時列出 `/volume1/apps`（五台）與 `/volume1/nfs-home`（原六台，含既有 `.206`）。

# 實作要點

- 建立 share：HTTPS `SYNO.Core.Share` `create`，`name` 與 `shareinfo` 用 JSON。此版 DSM 的 `validate_set` 可能回 3300，但 `create` 仍可成功；以後續 `get` / File Station / `showmount` 為準。
- NFS 規則：`SYNO.Core.FileServ.NFS.SharePrivilege` `load` 讀、`save` 整份 `rule` 列表覆寫。只對目標 `share_name` 呼叫 `save`。
- `root_squash=root` 對應 GUI「No mapping」，與現役 `nfs-home` 相同；不要改成 all-squash。
- 不要用 `192.168.1.201/29` 當 apps 匯出：會把 NAS `.200` 與未列入的 `.205`/`.206` 一併納入。
