---
title: ServerSideMap 1.3.14 Steam Cloud world save path failure
scope: projects/valheim-qol
status: verified-with-limits
updated: 2026-10-10
---

# 雲端世界的共用地圖附檔寫入失敗

SWOP / Valheim 1.0.17 / ServerSideMap 1.3.14 實機，Steam Cloud 新建測試世界 `CLITest1010`。遊戲主體顯示 `World save (5/5) done` 後，`ServerSideMap.SaveWorld+ZnetPatchSaveWorldThread.Postfix` 拋出 DirectoryNotFoundException，試圖寫入 `E:\worlds\CLITest1010.mod.serversidemap.explored`。雲端 GetDBPath 回傳 `/worlds/CLITest1010`，模組當成本機檔案路徑使用，附檔目錄不存在；主體存檔已完成，不代表共用地圖附檔成功。

已驗證處理：登出 → Manage saves → Worlds → 只選測試世界 → Move to local → 重新載入再存檔。主體存檔完成，擷取的新日誌無新例外，`%USERPROFILE%\AppData\LocalLow\IronGate\Valheim\worlds_local\CLITest1010.mod.serversidemap.explored` 已寫入並更新，4,194,316 bytes。未修改模組 DLL，未移動正式世界。

這只證明此版本／環境的雲端路徑問題與本機存檔處理有效；沒有第二玩家，不可判定先前隊友地圖同步問題必定同因，也未證明多人同步成功。下次先核對服主存檔位置與相同例外，再決定是否搬移正式世界並保留備份。
