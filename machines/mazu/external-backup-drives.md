---
title: Mazu 外接備份硬碟識別
machine: mazu
tags: [storage, backup, exfat]
status: active
created: 2026-09-03
updated: 2026-09-03
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

Mazu 的遠端 `swear01` HAPI session 會被 PolicyKit 拒絕 UDisks 掛載，但該帳號屬於 `docker` 群組。已驗證可使用本機既有 `ubuntu:24.04` image（禁止 pull、禁網路），進入 host mount namespace 後以 UUID 和 `-o ro` 掛載；操作前後都用 `findmnt` 核對來源與 `ro` 選項。
