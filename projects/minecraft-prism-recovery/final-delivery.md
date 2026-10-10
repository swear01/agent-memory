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

使用者已接受目前包的大小，不繼續削減快取、小地圖影像或內嵌 JAR。正式交付位於 `<Drive-root>/SomethingWouldDisappear/`：

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
