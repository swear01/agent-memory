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

用官方簽署的 OpenSpeedy 3.3.20 在同一個 Shelldiver PID 上加速。先讓遊戲載入 `speedpatch64.dll`，再設定 4 倍並啟用。初次反序時，設定指令結束後共用倍率回到 1，只有啟用位元。以 DLL 路徑、啟用位元、共享倍率和外部秒錶一起驗證。遊戲設為 60 FPS 時，島上基準約每秒 60.6 步，加速後約 236.6 步。另一個背景樣本在前景程序保持相同、Windows 游標座標未改變時，CLI 使島上玩家從 x3382 移至 x3260，牆鐘約 0.374 秒。原速單隻水母交易從店旁啟動，到原生存檔多 10 枚貝殼為 10.315 秒。

100m 的正常遠征結束有兩個視窗。先在潛艇範圍內互動，等動畫後用遊戲內座標 (0.4375, 196/360) 確認，再以按下前後的原生存檔雜湊區分確認視窗與結果畫面，最後按結果頁 (0.354, 0.732) 回島。水下氧氣仍大於零也可能已到結果畫面。回島後等角色與潛艇原生事件更新，再從固定的潛艇出發點 x3200 靠近登船。全程只送遊戲程序內輸入。

Invoke-ShelldiverCampaign.ps1 每批連續下潛，背包滿或最後一趟才在同一次到店行程賣水母。商店處理完排隊水母後，一次走到箱子並重複原生領款，直到原生存檔的待領款為零；長按一次可能只領一隻的收益。最後開一次技能樹，用實際餘額排可負擔的升級，逐一讀回等級與扣款。同一遊戲程序實測 OpenSpeedy 4 倍，兩批共四趟與七次原生升級耗時 34.7 秒，兩批結算的箱子皆為零。這段計時從已啟用加速開始，另一次從停用切換到 4 倍耗時 5.9 秒。

採廢料先在原生技能樹購買電漿槍。使用 CLI 的遊戲內世界座標 aim，靠到資源約 38 像素內，將原生 use_plasma_gun 持續 200 遊戲幀。短按或只模擬右鍵在背景測試未能穩定採到廢料。一次 100m 批次兩趟實測由 1 枚增至 22 枚廢料，原生存檔核對成功。tool_plasma_unlocked 是另外的存檔欄位，購買動作實際設定 is_plasma_tool_available。

原生購買滿 50 個技能等級後，200m 選項在 100m 右側 50 GUI 像素，已實測按 (.452125,.488)、沿用確認鍵 (.278,.645) 並進入 rm_ocean_shallows_02。此房間寬 1280，沿用活的潛艇回程定位。4 倍時暖態三批共 12 趟 200m，從進入第一批到所有結算保存共 79.0 秒，增加 112 貝殼。該樣本的漁獲隨生成而變，兩趟先導試跑沒有收穫。
