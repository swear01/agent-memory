---
title: Timber Rush 高倍速啟動的 Steam 成就與雲端存檔回退
scope: tools/openspeedy
tool: openspeedy
tags: [timber-rush, steam, achievements, cloud-save, unity-mono]
status: active
updated: 2026-11-28
---

適用於 Timber Rush Windows Steam build 26525431、OpenSpeedy 3.3.20。高倍速自動套用時，原版與 Auto Upgrade 補丁版均出現 SteamAPI.Init failed，本機成就仍繼續累積。從 OpenSpeedy 的記憶程序清單移除 Timber Rush，重新啟動 OpenSpeedy 和遊戲後，Steam 元件正常載入，遊戲開始送出已達門檻的成就。先查該遊戲的 Player.log、Steam stats_log 和加速模組是否隨啟動載入，再決定是否動遊戲程式。 [OpenSpeedy 3.3.20 程序清單](https://github.com/game1024/OpenSpeedy/blob/3.3.20/src/store/process.ts)

恢復 Steam 連線前，先備份本機 game_save.json 和 .bak。此版優先比較 savedAtUnixSeconds，再比較 saveRevision。高倍速影響 UTC 取時，雲端較低修訂的存檔可能有更晚的未來時間戳，並在首次成功連線時蓋過本機較高修訂的進度。先查清楚哪份是玩家最後進度，再使用未加速時鐘校正存檔中繼資料。先保存兩份，再透過該遊戲的 Steam Cloud 正常寫入管道還原。核對進度內容與備份相同，並等待 Steam cloud_log 顯示 Upload complete, result OK，再重開並確認沒有回退。 [Steam Cloud FileWrite](https://partner.steamgames.com/doc/api/ISteamRemoteStorage#FileWrite)
