---
title: Minecraft Prism 復原最終交付與清理
scope: project
project: minecraft-prism-recovery
status: completed
updated: 2026-10-10
tags:
  - minecraft
  - prism
  - recovery
  - google-drive
---

# 最終交付快照（2026-10-07）

此段為 2026-10-07 歷史決定；2026-10-10 已依新授權再次精簡，現況見下方。當時使用者已接受包的大小，不繼續削減快取、小地圖影像或內嵌 JAR。正式交付位於 `<Drive-root>/SomethingWouldDisappear/`：

| 資料夾 | 內容 | 包檔容量 |
|---|---|---:|
| `minecraft mod instances` | 31 份客戶端 `.mrpack` | 3,897,577,633 bytes（3.90 GB） |
| `minecraft maps` | 20 份原版／冒險地圖 ZIP | 1,109,034,257 bytes（1.11 GB） |

另有 47 份說明與驗證文件；98 個雲端檔案合計 5,014,095,996 bytes。這是已驗證的交付快照，後續若有新操作須重新讀取當時雲端清單，不視為永久即時數字。

## 復原與驗證範圍

- Prism 用「新增實例 → 匯入」開啟 `.mrpack`；原版地圖依各 ZIP 的 `RESTORE.txt` 選擇 Minecraft 版本並放入 `saves`。
- 由 server pack 整理的交付已轉成 client 結構；歷史檔名含 Server 不代表現行仍是伺服器包。
- 保留正式主要世界最新版本、config、操作設定、自訂模組與圖示。已接受的地圖裁剪結果不再重做；小地圖及作者完整冒險地圖沿用既有保留範圍。
- 可重新取得的 JAR 必須與已存版本和完整雜湊一致，不能換成最新版或同名檔；沿用 CurseForge／Modrinth 及使用者已授權的 GTNH 官方歷史參照。
- 51 個包在改資料夾前後均核對唯一 Drive ID、大小及 MD5，包內容沒有改動。這不代表每個遊戲或世界已實測；不例行重新逐包匯入、啟動或進世界，只針對具體錯誤查驗。
- Space Astronomy 已接受地圖損毀狀態；最新存檔的玩家與任務資料保留，不再追查已缺失的家與工廠地形。

## 已授權完成的清理

- 舊 `Minecraft instance` 的 382 個檔案、35,718,596,792 bytes 已移入 Drive 垃圾桶，未清空垃圾桶。
- 舊 `New Minecraft instance` 已重新分類；其文件移入 `minecraft mod instances/recovery-info`，原路徑不存在。
- 本次整理工作目錄 `work/minecraft-consolidation-20261003` 已整個清除，包括舊交付副本、下載材料、測試環境與本機成品副本；刪除前約 110,360,416,256 bytes。
- 本機僅保留 `outputs/minecraft-final-delivery-20261007` 約 7.5 MB 的文件與清單，以及 `state/` 的回條。不要再依賴已刪除的工作目錄或重開完成佇列。

## 後續查證入口

- `<project-root>/state/minecraft-final-folder-cleanup-20261007.json`：最終清理狀態、51 包雜湊與 Drive ID、移動及本機刪除結果。
- `<project-root>/state/minecraft-prism-current-cloud-inventory.json`：最後回讀的雲端清單。
- `<project-root>/outputs/minecraft-final-delivery-20261007/recovery-info/final-folder-cleanup.json`：小型交付摘要。
- Drive `minecraft mod instances/recovery-info/restoration-index.json`：package 欄位相對於 SomethingWouldDisappear。其他舊報告中的 53／54 包數、容量和路徑屬歷史證據，不能當現況。

## Vault Hunters S4 裁剪存檔（2026-10-10）

- 使用者確認語音中的 Bot Hunter 是 `Vault Hunters S4`，來源為 utux 的 Vault Hunters Remastered。此次「壓縮地圖」包括沿用先前的地形區塊裁剪；允許清除無需保留的地形及過期 Vault 地形，但 Vault 歷史與進度紀錄必須完整保存。這是對 S4 的新要求，不沿用上方 2026-10-07 停止縮減的決定。
- 正式包為 `SomethingWouldDisappear/minecraft maps/Vault Hunters S4 - 20261010.mrpack`；模組世界也放進使用者指定的 maps 資料夾。世界快照時間為臺灣 2026-10-10 14:41:58；無在線玩家時正常停服、完成存檔再複製與重啟。原伺服器未裁剪，既有 daily／weekly 備份不變。
- Minecraft 1.18.2、Forge 40.3.11、Java 17；160 個精確模組下載參照，156 個 server mods 與 client 同名檔逐一雜湊吻合，另有 4 個 client-only mods。不要用最新版本取代；Prism 匯入需連網下載。客戶端資源與操作設定保持原檔，gameplay config 以伺服器為準，衝突版本另留於 recovery-info。
- 沿用歷史裁剪程式，保護出生點、玩家位置、床位、可辨識的建築／機器／已用倉庫及玩家實體。此次主世界、地獄、終界均保留 8 區塊建築緩衝，未縮至 6／4；長時間活動區另有緩衝。
- 地形區塊數：主世界 58,237 → 6,525，地獄 11,397 → 1,017，終界 19,618 → 560。世界原始資料 642,645,925 → 95,817,637 bytes；封裝 519,215,723 → 155,320,363 bytes，縮小 70.1%。
- `data/the_vault_VaultSnapshots.dat` 的 `snapshot_refs` 有 85 筆；對應 `data/vault_snapshots/*.dat` 的 85 份 NBT 全部可讀、無缺檔，索引與快照逐檔 SHA256 不變。`the_vault_Vaults.dat` 活動 Vault 及 `the_vault_VirtualWorlds.dat` entries 均空；此快照的歷史 Vault 維度已無 region 地形可刪，沒有刪除任何 Vault 紀錄。
- 249 個非地形檔案與 9,412 筆保留的 terrain／entities／POI 紀錄核對 SHA256 不變；封裝排除暫態 session.lock。ZIP CRC、5,212 個成員雜湊、所有未改客戶端資源及 160 個下載參照均通過；Drive 完整讀回 155,320,363 bytes 的 SHA256／MD5 相符。
- 正式包 SHA256：`5d50c25847e13c7759eaa777cf2643b318d2d5f7e02d04e18e9d2f7ec185dd32`；MD5：`ec3a762d3770561452742ee32a53b1f0`。RESTORE、restoration、verification、backup-report、trimming、maps README 與共用 restoration-index 已同步並逐檔讀回；原有 20 份地圖 ZIP 的 ID／大小／MD5 未變。上方 2026-10-07 包數與容量只代表歷史快照。
- 完整 519 MB 原包及前版文件仍在 `<S4-task-root>/work/full-before-trim`，原始世界在 `<S4-task-root>/work/server-snapshot`；這些是本機保留副本，不是另存的完整雲端包。S4-task-root 為 `<user-documents>/Codex/2026-10-10/wduce-utx-bot-hunter-server-minecraft`；證據包括 `work/final-trimmed-verification.json`、`work/cloud-trimmed-full-readback.json` 與 `outputs/Vault Hunters S4 - trimming.json`。
- 未進遊戲實測；建築辨識為啟發式，可能漏判人工方塊，裁掉區域再次探索會重新生成。需要舊地形時取回完整原包；不可把雜湊完整性宣稱為所有建築或遊戲功能已驗證。

## 電腦備份清理與精簡包覆核（2026-10-10）

本次新授權要求可重建的模組以**原版本、原位元組的下載參照**保存，同時保留設定、操作按鍵及繁中語言包。這取代上方 2026-10-07 停止縮減內嵌 JAR 的決定；不代表授權刪除既有歷史世界。

- 兩套 Drive 電腦備份根目錄 `My Computer` 與 `我的電腦` 均已確認 `trashed=true`、`explicitlyTrashed=true`；TJ TEST 已另移垃圾桶，未納入本次設定備份。未操作 SWOP／M5 使用中的實例，未清空垃圾桶，不能宣稱 Drive 配額已釋出。
- `document/Setting Backup/app setting/Minecraft_Prism_設定精簡包_20261010.zip` 為 3,608,743 bytes，含 56 份原生 Prism 實例設定 ZIP，皆通過 CRC／結構檢查且沒有 JAR、世界或 TJ TEST。這是設定補充包，不能單獨恢復全部模組；原生 ZIP 匯入會重設遊玩時間，原始記錄另存。兩個 Opolis 來源缺少 cfg，已明示產生最小 OneSix cfg，沒有捏造遊玩時間。同名設定包有兩份雲端副本，大小／MD5 相同，本次未去重。
- 新增 `minecraft mod instances/SkyFactory4-4.2.4-Prism-compact-20261010.mrpack`：253,185,979 → 36,848,120 bytes；Minecraft 1.12.2／Forge 14.23.5.2860／Java 8。203 個下載項目均完整下載並核對 SHA256／SHA512；1,437 個保留成員核對來源 SHA256。保留設定、資源、兩份繁中語言包，不含世界。原生 cfg／mmc-pack／圖示放在 `overrides/recovery-info/original-native-instance/`，Prism 專屬設定與遊玩時間必要時手動還原。
- 新 SkyFactory 4 保留唯一已證實修改的 `SkyOrchards-0.0.12.jar`（80,532 bytes）：與官方同版本 81,239 bytes 相比有三個 class 改變。來源 SHA256 為 `ea07b5baf5e0c33b6079b946b1da4664470fecfe658ca2804824aaac19cd5e9c`，官方為 `fa0d780ca747bcc7e3ba6366529d41125b4b93ddf68e4bf6a72248fc3cb229b3`；不能因檔名／版本相同就替換。舊 SkyFactory 4 4.0.8 的同名 JAR 恰與官方相符，已改為下載項目，兩者不能混用。新包 SHA256 為 `1c25142c3ff649f9d67edf187c9a310befe08d7411c676694abcf1a0a84db068`。
- 盤點原有 31 個 MRPACK，更新其中 11 包，將 41 份內嵌 JAR 改成經完整下載雜湊驗證的參照。舊包仍有 **23 包／342 份 JAR（282 個不同 SHA256）**未取得精確下載證據，原檔保留；不能一律稱為自訂模組，也不能記成全部已符合零 JAR 規則。公開 CurseForge 歷史查詢遇到 403、GTNH 歷史目錄遇到 404，這僅代表查詢受限，不能證明檔案不存在。
- `minecraft mod instances` 現有 32 個 MRPACK（原 31 包加新 SkyFactory 4）。Vault Hunters S1／S2／S4 均無內嵌 JAR，下載項目分別為 151／167／160；S2 最後兩個 `unobtainium-1.7.0.jar`、`vhapi-3.4.0.jar` 已轉為精確下載。S4 位於 `minecraft maps`，本輪僅覆核，未修改另一任務的世界裁剪結果。
- 原 31 包的歷史世界壓縮後約 2.69 GB，本輪保持不變；改用下載清單不代表每包只剩數 MB。更新包通過 ZIP CRC、未改成員壓縮內容 SHA256 及 Drive 完整讀回 SHA256，之後才將 11 個被替換舊包及新 SkyFactory 4 的舊完整 ZIP 移到可恢復垃圾桶。沒有例行匯入 Prism、啟動遊戲或進入世界；完整性證據不等於遊戲實測。
- 雲端 `recovery-info/restoration-index.json` 已保留 S4 並增補新 SkyFactory 4 至 56 筆；README、例外清單與盤點報告均更新並完整讀回。本輪暫存 originals／updated／updated-final 合計 2,855,046,673 bytes 已刪，證據與交付保留；不要重開已完成佇列或依賴已刪暫存路徑。

查證入口：`<user-documents>/Codex/2026-10-08/new-chat/outputs/minecraft-compact-standard-audit-20261010.{md,json}`、`outputs/minecraft-computer-backups-trash-20261010.{md,json}`；同任務 `work/minecraft-compact-standard-20261010/` 下的 `cloud-replacements.json`、`updated-package-proofs.json`、`skyfactory-compact-proof.json`、`remaining-exceptions.json`、`documentation-update-proof.json`、`audit-report-cloud-proof.json` 與 `preset-cloud-copies-proof.json`。上方 Oct 7 的包數、容量及清單入口僅是歷史快照。
