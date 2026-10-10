---
title: ValheimCli 0.2.0 game-internal character controls and reliability limits
scope: projects/valheim-cli
status: implemented-runtime-unverified
updated: 2026-10-10
---

# 人物控制：改用遊戲內部方法

使用者希望 Agent 完整遊玩自己的角色，並明確要求「用遊戲內部方法，不應該用模擬鍵鼠」。最新 0.2.0 source head `e5c24fb233392a11bfec0bfb4173154acea6edcd` 已移除 WindowsInput.cs / SendInput，不能再沿用舊 head f77bf26 的按鍵／mouse 操作說明。

原始碼 GitHub `swear01/valheim-cli`，PR https://github.com/swear01/valheim-cli/pull/2 ，同一 concern branch `feat/character-control`。續作先查 PR／main／工作樹 live state，不要假設舊 pending 狀態仍成立。

## 控制架構與介面

- Harmony prefix 修改原版每次 `Player.SetControls` 的參數，保留原方法，不在 Plugin.Update 單次送控制值；否則下一物理更新會被 PlayerController 覆蓋。
- `input --move-x -1..1 --move-z -1..1 --actions attack,secondary,block,jump,crouch,run,dodge --ms 200 --confirm`。方向相對視線，斜向正規化；50–5000 ms；attack/block 有 press/hold，jump/crouch/dodge 一次觸發。蹲下是切換狀態，停止不會撤銷已發生的動作。
- `look --yaw 30 --pitch -10 --confirm` 直接呼叫 SetMouseLook，使用角度增量：正 yaw 向右、正 pitch 向下。可在 lease 中轉視角，不延長 lease。
- `action --action interact|slot|inventory|build-menu|hide|guardian|place|rotate` 走原版方法；slot 需 1..8，rotate 需非零 scroll ±10。place 設 m_placePressedTime 並進入原版 UpdatePlacement，保留材料、體力、技能與 cooldown 檢查；不要 raw TryPlacePiece/PlacePiece，它們本身不完成扣材料／體力流程。
- `ui --ui-action click|scroll --pointer-x 0..1 --pointer-y 0..1` 透過 Unity EventSystem.RaycastAll + ExecuteEvents。僅派給遊戲 Canvas，沒有 OS 游標或鍵鼠注入。電腦背包搬物品用點物品、再點目的地；沒有拖曳協定。
- 舊 --keys/--buttons/--mouse-x/--mouse-y/mouse 已從尚未公開的 0.2.0 移除。CLI／DLL 必須一起更新。
- AllowControl 預設 false，要求自己擁有的本機角色、活著、前景、沒有聊天／文字／主選單等。移動／戰鬥／look 被 inventory/map/store/build selector 等 UI 阻擋；UI 動作需相應 UI 開啟。普通控制可加入者使用，不需 OP；teleport 仍 host-only 且獨立 AllowTeleport。
- F12 撤權，stop 由網路執行緒只取消 managed lease；20 ms watchdog 不碰 Unity。下一遊戲控制 tick 送 neutral frame，再恢復人工輸入。處理 toggle block 與既有 autorun；切換人物、失焦、撤權、stop invalidates queued controls。卡住的 Unity thread 恢復前不能更新人物狀態。
- 固定最多 65,536 個 started/uncertain 寫入 ID，達限要求重啟（每秒一筆約 18 小時）；此限制未改。不能自動重送結果未知的寫入。

## 驗證邊界

本輪 bridge build、C# game/Harmony/UI fixture、9 項 Node tests（含 Node → production C# TCP 與新 write operations 防重送）、Windows/Ubuntu CI、打包與模擬 profile 安裝雜湊均通過。實際 1.0.17 assembly_valheim.dll 的獨立編譯 0 warnings/errors；metadata 核對私有欄位、Harmony 參數名稱、產物無 P/Invoke/WindowsInput。compile-only UI reference 新增 Unity3D.UnityEngine.UI 2018.3.5.1，排除其舊 UnityEngine 依賴；不隨包發佈，遊戲提供 runtime。

使用者明確選擇「先保留目前遊戲，完成程式與自動測試」；未重開 SWOP 遊戲、未安裝新 DLL 至正式 profile、未操作人物。以上不是實際 Harmony patch、UI handler、GPU 截圖、多人、耐久或自主通關證明。內部私有方法／欄位可能因更新改變，必須安排獨立測試 profile、角色、世界後再按 docs/agent-play.md 實測。

公開 Thunderstore 仍 ValheimCli 0.1.0；這份 0.2.0 未發佈 npm registry、Thunderstore 或 GitHub Release。先前 0.1.0 的短程實測不替代新功能驗證。只在本機研究實際遊戲 DLL／反編譯，不能提交或公開。

## 交付位置

`<project-root>/outputs/ValheimCli-0.2.0-experimental.zip` SHA256 `96f8241446a92decc725cd43134f614a257d4cf21eb61f69c35d7980e12596ef`。
`<project-root>/outputs/valheim-agent-cli-0.2.0.tgz` SHA256 `34a9be5979599753f59108974dd1af9dbbc86d928e2b0b73cb7d90e89fdccc1c`。
中文操作方式與當次完整驗證記錄為 `<project-root>/outputs/ValheimCli-0.2.0-操作方式.md`、`<project-root>/outputs/ValheimCli-0.2.0-驗證結果.json`。project-root 是 2026-10-08/new-chat-4 工作目錄。
