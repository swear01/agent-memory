---
title: DVLab fleet persistent storage migration inventory
scope: machines/hapi-fleet
project: dvlab-storage
tags: [storage, migration, backup, cleanup, fleet]
status: active
created: 2026-08-26
updated: 2026-09-29
---

# DVLab 儲存搬移與精簡保留

## 先判定是不是同一份資料

- NAS `192.168.1.199` 和 `.200` 共用同一份 `/volume1/nfs-home`。不同主機、IP 或 mountpoint 都不能算兩份備份。先用 `findmnt`、裝置 UUID、canonical export、inode 和權限分清現役 home、舊本機 home、scratch 與冷備份。舊路徑可能已改成指向現役 NFS 的 symlink，盤點勿跟入。用管理權限補查不可讀範圍及 `df`／`du` 差額；Snap 硬連結重算須核對 inode 和實際 blocks。
- 以內容雜湊找 exact duplicates，並確認同相對路徑、消費端引用和最終保留清單；只看大小、檔名或抽樣會誤刪。Valkyrie 兩個帳號的 `hw3/model.pkl` 曾同大小異內容，兩份均需保留。不能在 `.git/` 做 content hardlink，refs 的寫入可能互相污染。
- Canonical Home 以帳號身分建一個 `home/<account>/`。以現役內容為 base，G2/G3/G4 舊資料相異的內容用 `.archive/<generation>/` 保留，不覆蓋 current。同 numeric UID/GID 可能已分配新帳號，還原先核對 owner。Sealed overlay 只包含舊版必要差異，不能單獨當 standalone Home。需保存 account/disposition manifests。
- 預設保留 source、Git、README、論文、設計證據、最終結果、不可重建輸入。Dataset、模型、軟體或編譯輸出需有可取得來源及版本，不能因年代或目錄名決定可刪。Docker reclaimable、stopped 和 unreferenced 只描述引用狀態；若 volume 含研究環境，先用保留 owner/ACL/xattr/sparse 的 tar.zst 封存，做讀回、來源比對及跨主機 hash 確認。不要手刪 `overlay2`。

- Athena 的 `/` 與 `/mnt/md0` 曾是同一 ext4 NVMe，NFS `/home` 遮住的是本機舊 Home。先查同 UUID、子掛載與程序引用，避免把本機熱緩存發現成第二套備份。Hugging Face Arrow 處理 cache 與下載原料、解壓、模型分開盤點；只在逐項授權與可重建來源確認後清明確範圍。[Athena 2026-09-30 清理證據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/athena/storage-2026.md)
- NFS 單一 retired EDA 樹清理時，先核對帳號、舊路徑、現行共享庫和原有專案樹。已核准清掉 `jack0716/tsri.retired-20260914`，同帳號 `3dic` 研究樹保留。代表 VCS/cell-library 檔的 SHA 和 inode 驗證不足以宣稱整棵樹完全相同；比較部署與刪除前後作用域。[退休 EDA 來源核對與刪除收據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/nfs-jack0716-home.md)

## 搬移、故障碟與刪除閘門

1. 分批保存清單、容量、mtime、owner、SHA-256，查程序引用、special files、巢狀 mount、目的衝突和磁碟空間。NFS 可用 `rsync -aHAXS --numeric-ids` 建 incoming，讀回做帶 checksum 的 dry-run，核對 metadata，從第二個入口確認可讀。Docker 另查 container/builder 引用。SHA manifest 用相對路徑時從指定根目錄驗。
2. 目的地完全驗證後才升格，源目錄先改為 hold，檢查正式路徑與代表檔，再依該批授權清除。需要保留舊入口時驗證 symlink 指向相同 inode，使用者免保留舊路徑的例外不要套到別的帳號。以 `df` 記實際釋放量。別將歷史清理授權沿用到其他磁碟、分割區或帳號。
3. 冷備份須做解壓、成員／payload hash、完整讀回並依保留政策確認覆蓋後才能取代來源。截斷 archive、單靠 CRC／ISIZE、只有 delta 或 partial 都不足。未加密的歸檔排除 credentials、SSH/GPG identity、browser/history、Xauthority、editor/cache/runtime；個別明確授權的放置例外不外推。
4. Valkyrie `/mnt/md1` 曾實際 EIO；SMART overall PASSED 不能蓋過 UNC、pending、offline uncorrectable 及相對應的核心日誌。用有 sudo 的維護帳號驗 exact device 與 SMART，先保全來源，再由操作者安排替換。未換前不要為少量空間掃描、加負載或大量 unlink。歷史上的 EIO 與當下裝置健康要分開驗。

## 仍有用的恢復定位

- [Mazu 長備份流程](../mazu/long-backup-cleanup.md) 和 [外接碟識別](../mazu/external-backup-drives.md) 是冷碟、Canonical Home 與 Jonathan 的恢復入口。Jonathan 是獨立永久保護封存。2026-09-21 移到 One Touch 長碟，早期 Ultra Touch 位置已退役。
- 2026-09-29 Valkyrie 舊 Home 私有封存移至 One Touch 長碟 `backup/private-system-backups/valkyrie-g1-migration-20260926/`，原 NFS 私有路徑已撤。未加密，exFAT 掛載權限只限制本機。七個研究 Docker volume 封存在 NFS `<admin-home>/private-system-backups/valkyrie-docker-repro-volumes-20260929/`。原本未被引用的 volume 含非空研究資料。
- 舊 Zeus Home 的完整 Mazu SSD 包已依該批授權移除。補存少量程式碼與測試資料放長碟 `backup/private-system-backups/zeus-g2-unique-source-20260929/`，附 SHA manifest；只承諾精簡範圍。其餘來源依現役 Git/NFS 和 Canonical Home 成品查找。Cthulhu 的研究專案經確認搬入 canonical NFS，Valkyrie 某些 active local homes 仍須各自核對。逐帳號與來源範圍查原始驗證收據。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/storage-migration-2026.md)
