---
title: Minecraft Prism 復原最終交付與清理
scope: project
project: minecraft-prism-recovery
status: completed
updated: 2026-10-07
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
