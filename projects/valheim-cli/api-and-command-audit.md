---
title: ValheimCli official API and internal command mapping audit
scope: projects/valheim-cli
status: source-verified-tests-pending
updated: 2026-10-10
---

# 官方 API 與實際遊戲介面對照

使用者要求先確認官方 API 與自製模組是否對到遊戲內指令，更新 QMD 後再測試。保留原先不干擾 SWOP 正式遊戲的限制，本輪不啟動遊戲、不安裝、不操作角色。

## 官方文件

2026-10-10 查閱 Iron Gate 官方 FAQ、Regarding Mods 與 1.0 FAQ，仍表示沒有官方 mod support，更新相容性不保證。未找到官方公開並承諾相容的 Agent／角色控制 SDK 或遠端控制 API。BepInEx 是社群的 Mono plugin framework，不是 Iron Gate 的官方 API。

- https://www.valheimgame.com/faq/
- https://www.valheimgame.com/news/regarding-mods/
- https://www.valheimgame.com/zh/support/valheim-1-0-faq/
- https://github.com/BepInEx/BepInEx/wiki/Home

官方確有資產載入介面文件，例如 SoftReference<T>.Load / LoadAsync / Release 以及 Runtime.AddManifest；用途是 assets，不是角色行走／戰鬥。官方也說明 -console、F5、devcommands，但這是開發者 console，不能視為完整 Agent 操控 API。

- https://www.valheimgame.com/support/modding-faq-for-the-asset-bundle-update-0-217-40/
- https://www.valheimgame.com/support/how-to-enable-developer-mode/

## 0.2.0 實作與 Valheim 1.0.17 DLL 靜態核對

核對 head f77bf2687d34458484d94ecf6b7c66ec58dc36b0 與先前從 SWOP 取得的 assembly_valheim.dll；只在本機查看遊戲反編譯，不發布或提交遊戲 DLL／程式碼。

| CLI 操作 | 目前遊戲端實作 | 是 console 指令嗎 |
| --- | --- | --- |
| status / players | Player.m_localPlayer、ZNet、Player.GetAllPlayers、位置、血量與體力方法 | 否，直接讀取遊戲物件 |
| observe | GetInventory().GetAllItems() 與 Unity ScreenCapture | 否 |
| teleport | Player.TeleportTo(Vector3, Quaternion, bool) | 否，直接呼叫遊戲方法；不是執行 goto 字串 |
| input / mouse | Windows SendInput，後續由遊戲正常讀鍵鼠輸入 | 否；尚未直接接 Player.SetControls 或 ZInput |
| stop | 停止自己的輸入控制器、嘗試釋放鍵鼠 | 否 |

bridge 沒有 Terminal.ConsoleCommand 註冊、TryRunCommand、Harmony input hook 或任意 console execution。F5 內也不會因此新增 valheim / input / observe 指令；這些是外部 Node CLI 的操作。

1.0.17 的 Player.SetControls 實際存在，參數為 movedir、attack/attackHold、secondaryAttack/secondaryAttackHold、block/blockHold、jump、crouch、run、autoRun、可選 dodge。PlayerController.FixedUpdate 在 network owner 檢查與 TakeInput UI 檢查後，每個物理更新會讀 ZInput 並呼叫 SetControls；TakeInput=false 時會呼叫零輸入。因此不能只在 Plugin.Update 單次呼叫 SetControls 就宣稱穩定控制，原版下一個物理更新可能覆蓋該值。未實作這個替代介面；相機、背包、建造與文字輸入仍需各自核對。

本機 README 的走路／跳躍／閃避／攻擊例子明確假設預設按鍵。改鍵後須由 Agent 校準，不能視為已與語意操作綁定。原專案編譯 reference package 是 Digitalroot.Valheim.Common.References 1.0.16；本輪規劃用實際 1.0.17 DLL 在獨立暫存專案編譯，區分 API 簽章相容與真正載入遊戲的驗證。

## 下一步測試

此文件先同步並更新 QMD，再執行 C# fixture、Node tests、原 bridge 編譯與實際 1.0.17 DLL 參照編譯。結果尚未執行，不能把靜態方法存在當成遊戲實測通過。
