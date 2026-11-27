---
title: Shelldiver 的原生 CLI 與可見場景驗證
scope: projects/shelldiver
status: active
updated: 2026-11-27
---

Steam App 3862670、build 23898569、遊戲 v1.0.2.300.020 使用 GameMaker YYC。`data.win` 可核對房間和資產，但 UndertaleModCli 無法從中反編譯玩法程式。Aurie 2.0.2 搭配 YYToolkit 4.0.1 在這個 build 啟動時發生存取違規。YYToolkit 5.0.0c 發行 DLL 已實機正常載入，但它使用 `YYTK_ZeusMain`，發行標籤對應的穩定分支標頭仍使用 `YYTK_Main`。使用上游 `experimental` 的 `d5cc0078bfbfd5907884ad3fe0b3c6cbaa1c3f54` 標頭，並以 `NDEBUG` 編譯，Aurie 才能以 Release 載入橋接 DLL。

Steam 從 CLI 啟動後等標題頁的 Enter 提示可用。遊戲內的短脈衝只覆寫本程序查詢 Return 的結果；不要使用 GameMaker `keyboard_key_press()`，該 API 在 Windows 可能將按鍵傳到其他程式。Continue 直接呼叫原生 `o_button` 的既有 action，核對其識別為 `continue_button`，以及 `files_loaded`、`is_tutorial_finished`、`xbox_loading_user`、`transitioning` 四個狀態。實機同一存檔確實進入 `rm_main_island`。適用於所有遊戲的限制見[自動玩遊戲的操作與通關偏好](../../global/preferences/gameplay-automation.md)。

在 `o_input` 的 Begin Step 走完原生輸入更新後套用短脈衝，利用後續角色事件取得座標。島上可透過原生互動把背包水母交給 x3480 的商店，再到 x3375 的箱子領取。原存檔中五隻水母經原生銷售共取得 40 枚貝殼，已用原生 `Game.json` 核對保存。該檔是十六進位文字，解碼後略過 16 bytes 再讀 UTF-8 JSON。保留開始前的副本。

在可見島上，以角色輸入步數和真實時間取樣，240 FPS 設定約每秒 240.1 步；標題選單以 480 FPS 設定約每秒 480.4 步。數值描述步進，水下捕撈效率需另測。
