---
title: "Slay the Spire 2 模组排查：先看 godot.log 的 MissingMethodException"
scope: tools/games
status: active
updated: 2026-11-14
---

# Slay the Spire 2 模组排查：先看 godot.log 的 MissingMethodException

## Problem

macOS 上 StS2（Steam app 2868840）開局卡在 Neow 無法對話。原因是 Spirebird Stats（Workshop 3747979993，2026-06 後未更新）patch 了 `HoverTip.set_Id`，遊戲更新後該 setter 被移除；Neow 生成詛咒選項時讀 Cursed Pearl tooltip 觸發 patch，丟出 `System.MissingMethodException` 讓 `Neow.GenerateInitialOptions()` 失敗。

## Durable rule

- 卡關先 `grep -n "Exception" ~/Library/Application Support/SlayTheSpire2/logs/godot.log`；stack trace 裡的 patch 類別（例如 `CardStatsMod.Patch_*`）直接指出是哪個模組。舊 log 會輪替成 `godot<timestamp>.log`。
- 模組啟用狀態在 `~/Library/Application Support/SlayTheSpire2/steam/<steamid>/settings.save` 的 `mod_settings.mod_list`（`id`、`is_enabled`、`source`）；遊戲關閉時改、先備份。同一模組本地 `mods/` 與 Workshop 各一份時會有兩筆，遊戲只載一份。
- Workshop 模組位置：`~/Library/Application Support/Steam/steamapps/workshop/content/2868840/<id>/`；本地模組在 `SlayTheSpire2.app/Contents/MacOS/mods/`。
- 勝率／統計類替代：Spire Codex（Workshop 3747536911）較新且最多人用；未登入時 `upload=False`，附帶的 damage meter 預設開，會與其他傷害統計模組重疊，要在遊戲內 BaseLib 模組設定關。
- Workshop 訂閱數查詢用 Steam 公開 API `ISteamRemoteStorage/GetPublishedFileDetails/v1/`（POST `itemcount`、`publishedfileids[i]`），steamcommunity 網頁 WebFetch 容易 429。
- Steam 客戶端視窗無法用 computer-use 控制（helper process 無法授權）；訂閱／取消訂閱改用內建瀏覽器登入 steamcommunity.com 操作，`#SubscribeItemBtn` 有 `toggled` class 表示已訂閱。Steam 會自動下載或刪除檔案。

## Verification

2026-11-14：停用並取消訂閱 Spirebird 後，godot.log 顯示 Spire Codex v1.4.0 載入，使用者回報 Neow 恢復正常。
