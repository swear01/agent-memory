---
title: Prism 復原：世界裁剪、精確模組下載與 Drive 清點
scope: project
project: minecraft-prism-recovery
status: completed
updated: 2026-10-11
tags:
  - minecraft
  - prism
  - recovery
  - google-drive
---

- **找不到最新包，或以為名字含 Server 就要跑 server。** 用 Drive `SomethingWouldDisappear/minecraft mod instances/recovery-info/restoration-index.json` 定位，索引內 package 相對於該根。Prism「新增實例 → 匯入」開客戶端 MRPACK；普通地圖依 `RESTORE.txt` 選版本、解至 saves。S4 世界包在 maps（Minecraft 1.18.2／Forge 40.3.11／Java 17）；SkyFactory4 4.2.4 compact 是另一包（1.12.2／Forge 14.23.5.2860／Java 8）。歷史包數、舊清單和已刪 scratch 不當現況。
- **想將 JAR 換成同版本下載。** 下載完整成品並驗原 SHA256/SHA512；檔名和版本不夠。4.2.4 的 `SkyOrchards-0.0.12.jar` 有三個 class 與同版官方不同，須保留；4.0.8 的同名 JAR 才已驗可替換。403／404 只表示未取得證據，保留其他未能精確替代的 JAR。MRPACK 保留 config、操作、語言包和完整下載參照；設定精簡 ZIP 不含世界或 mods，Prism 原生匯入會重設遊玩時間，需另查原 cfg。
- **裁剪 S4 地形也順手刪了 Vault 歷史。** 在停服完整快照的副本操作，正式伺服器及既有備份保留。保護玩家、床位、建築、倉庫，以及 `the_vault_VaultSnapshots.dat` 索引、對應 NBT 與進度；核對所有保留的 terrain/entities/POI 與非地形雜湊。8 區塊緩衝及建築辨識是啟發式，無法保證無漏。需原地形時找 `<S4-task-root>/work/full-before-trim` 或 `server-snapshot`，本機副本並非第二份 Drive 完整包。CRC、雜湊與 Drive 讀回不等於已進遊戲驗證。
- **祖先資料夾進垃圾桶，但 active 容量不降。** 查子檔自身的 `trashed` 欄位。新授權清理時按檔案 ID、現行名稱、父目錄、大小／MD5 確認，再逐一讀回與 changes 清點。Trash 未永久清空時不宣稱帳戶配額已釋出；維持歷史世界及現用 SWOP／M5 實例。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/projects/minecraft-prism-recovery/final-delivery.md)
