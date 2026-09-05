---
title: Mazu 外接備份硬碟識別
machine: mazu
tags: [storage, backup, exfat, nfs, acl]
status: active
created: 2026-09-03
updated: 2026-09-05
---

# Mazu 外接備份硬碟識別

以 UUID 識別，不依賴可能變動的 `/dev/sdX`：

- 長備份碟：Seagate One Touch，UUID `001D-7DC1`，約 4 TB。`backup/` 內是按來源命名的 `.tgz` 靜態歸檔（如 `dvlab.tgz`、`ntuwp.tgz`、`ric.tgz`），時間集中於 2025-06-03。
- 短備份碟：Seagate Ultra Touch，UUID `84CE-326E`，約 5 TB。`backup/<remote-home>/`
  內主要是 2025-06 建立的逐帳號 `.tgz` home snapshots，不是展開後可直接瀏覽的 home tree。

2026-09-03 在 Mazu 驗證兩顆皆以唯讀模式掛載：

```text
/dev/sda2 /mnt/one-touch   exfat ro
/dev/sdb2 /mnt/ultra-touch exfat ro
```

## 實際使用量

- 長備份碟完成精簡後，filesystem used 465,195,237,376 bytes（433.247 GiB）；
  根目錄只剩 `backup/`。
- 短備份碟在加入 Jonathan canonical archive 後，2026-09-05 實測 filesystem used
  1,884,933,062,656 bytes（1.885 TB decimal／1.714 TiB）。原有 45 個 `.tgz` 合計
  1,808,648,730,588 bytes（1.645 TiB）；另有 76,204,875,019-byte 的
  `jonathan.tar.zst` 與 83-byte SHA-256 sidecar。「短備份」描述保留週期／用途，不代表容量較小。

因此長備份碟目前是四百多 GB allocated；短備份碟實際保存約 1.885 TB。

## 2026-09-05 長備份提交後狀態

長備份碟已以兩份完整讀回驗證的精簡 archive 取代原內容，並依使用者要求保留原始路徑與檔名：

- `backup/dvlab.tgz`：178,168,055,053 bytes，SHA-256
  `301251df790c54eb50b40816fc1d478c12b1f6a63ac2f60b3c56ee93720a0d14`。
- `backup/yoctol.tgz`：236,762,429,270 bytes，SHA-256
  `35ac9e0424be2ea2cabee021cda915643ae1abcb05e891e2214a99f656af0893`。

舊兩檔只在兩份成品完成本機與外接碟全量驗證後移除；已驗證的 Trash、Seagate 出廠檔與
OS metadata 也已清理。2026-09-05 最後實測 filesystem used 465,195,237,376 bytes（12%），
根目錄只剩 `backup/`，掛載選項為 `ro`；詳見 `long-backup-cleanup.md`。

## 2026-09-05 短備份來源與處理狀態

- 45 個 `.tgz` 中有 42 個逐帳號 archive；每個第一個 tar root 都是同名帳號並保留當時
  UID/GID。42 個名稱目前全有現行共享 NFS home directory，其中 40 個仍可由 NSS 解析；
  `athena` 與 `b09901005` 只剩 directory、NSS 已無帳號。
- Mazu 現行 home 來自共享 NAS NFS，而不是本機系統碟。因此這批是 2025-06 的多使用者
  NFS home snapshots；Mazu 只是掛載與備份執行點，不能描述成 Mazu 本機某一目錄的單一鏡像。
- 其餘三個 `.tgz` 是 `#recycle.tgz`、`qsyn_benchmark.tgz` 與 272,522,182,543-byte
  `copy.tgz`。`copy.tgz` 對應 NAS 仍存在的舊 `copy/cthulhu_home` 搬家世代；完整串流曾在
  gzip trailer 回報 CRC error，不能視為健康備份或直接刪除。
- 原有 45 個 `.tgz` 目前只做過 inventory／來源判讀，尚未整批重製或刪除。唯一完成精簡、
  寫入與完整讀回驗證的是 `backup/<remote-home>/jonathan.tar.zst`；它整合現行 NFS 與
  Valkyrie legacy home，是永久保護的唯一 canonical Jonathan home，不是 2025-06 舊 snapshot。

Mazu 的遠端 `swear01` HAPI session 會被 PolicyKit 拒絕 UDisks 掛載，但該帳號屬於 `docker` 群組。已驗證可使用本機既有 `ubuntu:24.04` image（禁止 pull、禁網路），進入 host mount namespace 後以 UUID 和 `-o ro` 掛載；操作前後都用 `findmnt` 核對來源與 `ro` 選項。

## 共享 home 的 ACL 限制

備份 handoff 若位於 `swear02` 的 `<remote-home>`，Mazu 上應直接使用 owner
帳號 `swear02` 讀取與處理；不可因一般 Mazu HAPI runner 使用 `swear01`，就改以
`swear01` 存取或替它設 ACL。只有使用者明確要求跨帳號分享時才處理 ACL。

2026-09-04 實測 Zeus 與 Mazu 的共享 home 都由同一個 NFSv4.1 backend
提供。客戶端 `setfacl` 回覆 `Operation not supported`，`nfs4_getfacl` 也回覆
`Operation to request attribute not supported`；儲存伺服器的 SSH 連線則被拒絕。
因此目前無法從任一 NFS client 設定 named user ACL。需要先在儲存伺服器端
啟用／設定 ACL 或開放管理通道；在此之前只可用 owner-only mode，不能宣稱
已對特定使用者完成最小 ACL。

Zeus 與 Mazu 的 `/var/tmp` 是各自主機本機目錄，不會互相同步。若分析產物在
Zeus 的 owner-only 暫存目錄，而 Mazu runner 是另一帳號，可用已驗證的 SSH
host key 與 `BatchMode=yes` 連線，先確認 Zeus 既有 Docker image，再以
`--pull=never --network none --read-only --user 0:0` 和唯讀 bind mount 將指定小檔
串流到 Mazu 的本機 staging。全程不可停用 host-key 驗證；接收後必須先核對
來源公布的 SHA-256，再原子替換正式報表。這條路只適用於明確列名的交接檔，
不能拿來遍歷或複製另一帳號的其他資料。
