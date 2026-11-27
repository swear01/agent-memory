---
title: Mazu 外接備份硬碟識別
scope: machines/mazu
machine: mazu
tags: [storage, backup, exfat, nfs, acl]
status: active
created: 2026-09-03
updated: 2026-09-30
---

# 先用 UUID 定位冷備份，再讀回

| 角色 | 裝置識別 | 恢復定位 |
| --- | --- | --- |
| 長備份 | Seagate One Touch、4 TB、exFAT、UUID `001D-7DC1` | Mazu 接碟時為 `/mnt/one-touch`，`backup/home/Canonical-Home.repacked-20260915/` 有已發布 54 卷、`COMPLETE.json`、`SHA256SUMS`、`volumes.json` 和還原 README。 |
| 短備份 | Seagate Ultra Touch、5 TB、exFAT、UUID `84CE-326E` | 2026-09-21 已核對清空且安全卸載，早期 Jonathan 和 `copy.tgz` 位置已退役。 |
| 短備份 | One Touch w/PW、4 TB、ext4、UUID `937f406b-e6e0-421c-a3ed-fbb52652adc2` | 2026-09-21 依明確授權刪除對應 G4 副本，只剩空 `lost+found`。逐 byte 比對只做了 10/21，不能描述成全量。 |

- Mazu 是管理主機，冷碟、系統碟與工作 SSD 是三個不同 filesystem。用 UUID、filesystem、`findmnt` 和現場連接查證。舊 mountpoint 存在或兩顆碟曾同時在線不證明此刻都在。工作優先以 `/usr/2TB-SSD/backup-work` 完整批次落盤，先核對 ext4 mount／UUID 和可用容量，權限錯誤不等於空間用完。
- Jonathan 唯一獨立 canonical 為長碟 `backup/home/jonathan.tar.zst` 與 `.sha256`，不在 54 卷裡。2026-09-21 完整 SHA、解壓與 tar traversal 均通過後才移除短碟原本。任何後續替換都須確認逐 byte／hash／archive 全部可回讀並有人工作出授權。
- 長碟 `backup/private-system-backups/valkyrie-g1-migration-20260926/` 保存搬來的 50 份 archive 和 sidecars；`zeus-g2-unique-source-20260929/` 是精簡的程式碼與測試資料，不是舊 Zeus Home 全備份。這些未加密，Unix 掛載 ACL／mask 不提供跨系統保護。以 [fleet 遷移筆記](../hapi-fleet/storage-migration-2026.md) 核對保留邊界及個別收據。
- 來源 archive 的 CRC 不過並非刪除理由。舊 `copy.tgz` 的 G4 內容透過 account disposition、sealed overlay required manifest 和最終 54 卷收據做內容替代，只涵蓋當時接受的排除規則。欠缺證據的 partial 不當恢復來源，與已完成作品分開列。
- 有 writer 的 staging 可能含 SQLite、FIFO 和另掛的 NFS live home。先確認 process references、子掛載、ACL、inode 和 RW 界線；不要將大量連到現行 NFS 的檔案當成本機 payload。已停用 staging 搬移保留 metadata，完整比對後才切路徑。短暫寫冷碟之後 sync、讀回、恢復 RO，再安全卸載。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/mazu/external-backup-drives.md)
