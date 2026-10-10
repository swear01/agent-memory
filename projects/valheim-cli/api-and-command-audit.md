---
title: ValheimCli official API and internal action mapping audit
scope: projects/valheim-cli
status: static-api-build-and-fixtures-verified-runtime-pending
updated: 2026-10-10
---

# 官方 API 與實際遊戲方法

使用者要求核對官方 API、更新 QMD 後測試，後續再要求移除模擬鍵鼠。保留不干擾 SWOP 正式遊戲的限制。前次 f77bf26 API audit 已先同步 commit 5c5755a／QMD，再通過 fixture 與 actual-game assembly build；那份 SendInput 架構現已被 e5c24fb 取代。

2026-10-10 查閱 Iron Gate FAQ、Regarding Mods、1.0 FAQ：沒有官方 mod support，未找到保證相容的 Agent／角色控制 SDK。官方 SoftReference assets API 用於資產，不是角色行走／戰鬥。BepInEx 是社群框架，console 開發指令也不是完整 Agent 控制 API。

- https://www.valheimgame.com/faq/
- https://www.valheimgame.com/news/regarding-mods/
- https://www.valheimgame.com/zh/support/valheim-1-0-faq/
- https://www.valheimgame.com/support/modding-faq-for-the-asset-bundle-update-0-217-40/
- https://www.valheimgame.com/support/how-to-enable-developer-mode/

## 最新 e5c24fb 對照

| CLI | 遊戲內方法／入口 |
| --- | --- |
| status / players | Player.m_localPlayer、ZNet、GetAllPlayers、位置與血量／體力 |
| observe | GetInventory().GetAllItems()、Unity ScreenCapture |
| teleport | Player.TeleportTo，仍 host-only；沒有執行 goto 字串 |
| input | Harmony prefix 修改每次 Player.SetControls 參數，原版方法照常執行 |
| look | Player.SetMouseLook，角度增量 |
| action | Player.Interact（private）、UseHotbarItem、InventoryGui.Show/Hide、Hud.TogglePieceSelection、HideHandItems、StartGuardianPower、UpdatePlacement（private）、m_placeRotation（private） |
| ui | Unity EventSystem.RaycastAll／ExecuteEvents，遊戲 Canvas click/scroll |
| stop | 取消 managed lease／epoch，下一 game tick 中性控制 |

沒有 Windows SendInput、Terminal.ConsoleCommand 註冊、TryRunCommand 或任意 console execution。F5 不會新增 valheim/input/observe 指令，它們是外部 Node CLI 操作。

1.0.17 PlayerController.FixedUpdate 每次讀 ZInput 呼叫 Player.SetControls，TakeInput=false 也送零控制；因此單次 Plugin.Update 呼叫會被覆蓋。最新版本在原始呼叫處修改參數，並核對 movedir、attack/attackHold、secondaryAttack/secondaryAttackHold、block/blockHold、jump、crouch、run、autoRun、dodge 的實際參數名稱。ToggleBlock 會切換 m_blocking，不能只在結束時送 block=false；prefix 依目標 hold 狀態調整 toggle，並取消原有 autorun。

Placement 必須進原版 UpdatePlacement 驗證／扣材料／體力／技能／耐久路徑；TryPlacePiece 本身不能替代這整段流程。原版電腦 InventoryGrid.OnLeftDown 已支援點物品再點目的地，UI 不需要 OS drag。

## 已驗證範圍

原 production 1.0.16 references build 與替換成實際未 publicize 的 1.0.17 assembly_valheim.dll 獨立 build 都 0 warnings/errors。輔助 DLL、Unity/BepInEx/UI 參照沿用現有 compile-only packages，沒有聲稱全部 helper runtime 相同。metadata audit 確認所用 private fields、Harmony 參數名稱與產物沒有 P/Invoke/WindowsInput。C# fixtures、Node 9/9、Windows/Linux CI、打包／假 profile 安裝雜湊也通過。

這些不證明實際遊戲載入、Harmony 與其他 mod 並存、真實 UI、GPU、多人或耐久已通過。沒有重開、安裝或操作 SWOP 正式遊戲。最新介面、產物雜湊與完整限制見 character-control-0.2.0.md；當次 evidence 在 `<project-root>/outputs/ValheimCli-0.2.0-驗證結果.json`。本機實際 DLL／反編譯不可公開。
