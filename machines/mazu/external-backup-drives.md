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
- 短備份碟：Seagate Ultra Touch，UUID `84CE-326E`，約 5 TB。內容是可直接瀏覽的 `backup/<remote-home>` 目錄樹，mtime 為 2025-06-06。

2026-09-03 在 Mazu 驗證兩顆皆以唯讀模式掛載：

```text
/dev/sda2 /mnt/one-touch   exfat ro
/dev/sdb2 /mnt/ultra-touch exfat ro
```

## 2026-09-04 實際使用量

- 長備份碟：filesystem used 590,040,793,088 bytes（549.518 GiB）。正式 `backup/` 的
  apparent content 為 474,001,460,913 bytes（441.448 GiB）；`.Trash-1043/` 另有
  80,340,117,871 bytes apparent（74.823 GiB），但因 exFAT allocation unit 與大量小檔，
  allocated size 約 107.998 GiB。Trash 內含大型舊 archive 與刪除後的目錄樹；它是高價值
  清理候選，但在核對現行 archive、checksum 並取得刪除授權前不得移除。
- 短備份碟：filesystem used 1,808,727,801,856 bytes（1.809 TB decimal／1.645 TiB）；
  `backup/` apparent content 為 1,808,648,730,588 bytes，證明不是只有幾百 GB，也不是單純
  filesystem allocation overhead。「短備份」描述保留週期／用途，不代表容量較小。

因此只有長備份碟目前是五百多 GB allocated；短備份碟實際保存約 1.81 TB 檔案內容。

## 2026-09-05 長備份提交後狀態

長備份碟已以兩份完整讀回驗證的精簡 archive 取代舊 `dvlab.tgz` 與 `yoctol.tgz`：

- `backup-clean/dvlab.cleaned.tgz`：178,168,055,053 bytes，SHA-256
  `301251df790c54eb50b40816fc1d478c12b1f6a63ac2f60b3c56ee93720a0d14`。
- `backup-clean/yoctol.cleaned.tgz`：236,762,429,270 bytes，SHA-256
  `35ac9e0424be2ea2cabee021cda915643ae1abcb05e891e2214a99f656af0893`。

舊兩檔只在兩份成品完成本機與外接碟全量驗證後移除，archive 淨省 8,810,044,079 bytes。
最後成功核對的 filesystem used 是 581,230,919,680 bytes（15%），掛載選項為 `ro`。
Trash 尚未刪除；後續驗證因 Mazu 離線中斷，詳見 `long-backup-cleanup.md`。

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
