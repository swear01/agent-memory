---
title: ValheimCli 0.2.0 character control implementation and reliability limits
scope: projects/valheim-cli
status: implemented-runtime-unverified
updated: 2026-10-10
---

# 人物控制完成狀態與恢復工作位置

使用者希望 Agent 能完整遊玩一個角色。0.2.0 已加入 observe 畫面與背包、健康／體力／視角、有限時鍵鼠輸入、游標操作及 stop；這是控制工具，尚未證明自主導航、採集、建築或戰鬥任務能可靠完成。

原始碼：GitHub `swear01/valheim-cli`，PR https://github.com/swear01/valheim-cli/pull/2 ，branch `feat/character-control`，head `f77bf2687d34458484d94ecf6b7c66ec58dc36b0`。工作樹 `<worktree-root>/valheim-cli/character-control` 保留且乾淨。2026-10-10 查得 PR OPEN / mergeStateStatus UNSTABLE，Swear Review pending；Cursor Bugbot 已對此 head SUCCESS，依 OR review 政策不再觸發其他 review。Windows / Ubuntu CI 通過。這是狀態快照，續作須重新查詢，不得繞過合併檢查。

## 已驗證與未驗證

- net48 bridge 編譯 0 errors / 0 warnings；C# protocol、dispatcher、transport、控制政策與截圖生命週期 fixture 通過；9 個 Node tests 通過，包括 Node → production C# TCP fixture。
- 封裝、版本與雜湊、解壓後 CLI help、DLL 安裝至模擬 BepInEx profile 通過。fixture 使用假輸入與遊戲替身，沒有測真實 Windows 鍵鼠或 Unity 執行。
- 使用者明確選擇「先保留目前遊戲，完成程式與自動測試」；未中斷 SWOP 正式遊戲、未安裝 0.2.0 至正式 profile、未重開或輸入操作。真實遊戲、多玩家、焦點切換、DPI、戰鬥與長時間耐久均未驗證。
- 公開 Thunderstore 仍為 ValheimCli 0.1.0；0.2.0 尚未發布 Thunderstore、npm registry 或 GitHub Release。先前 0.1.0 短程 status / players / 自身傳送實測不能代替新控制功能的驗證。

## 架構檢查結論

本機 loopback + token、嚴格 request 驗證、主執行緒執行遊戲 API、寫入 ID 防重複、未知結果不自動重試，是合理基礎。AllowControl 預設 false，按鍵限時 50–5000 ms，20 ms 獨立 watchdog、F12 撤權、stop 與 epoch 防止舊排隊操作恢復。

人物控制目前依賴 Windows SendInput：WindowsInput.cs 的前景檢查與送出輸入並非原子操作，切換視窗仍有競態；實體按鍵可能干擾。InputController 的 watchdog 與遊戲同程序，強制終止程序後不能保證釋放鍵鼠。Server.cs 保留最多 65,536 個寫入 ID，達限會拒絕操作並要求重啟；每秒一筆約 18 小時，更密集滑鼠指令更快達限。以上限制尚未修改；改用遊戲內部輸入介面只是後續評估方向，未實作或證明相容。

下一步應先由使用者安排不影響正式遊戲的實機時段，使用複製 profile 與可丟棄角色／世界，完成 docs/agent-play.md 的 live acceptance checklist；不要直接宣稱完整 Agent 遊玩可靠。

## 交付位置

`<project-root>/outputs/ValheimCli-0.2.0-experimental.zip` SHA256 `96bb136a1b069dc5e7b7516b8e21ff80d5de0ee948b32b38abd4be85af454c42`。
`<project-root>/outputs/valheim-agent-cli-0.2.0.tgz` SHA256 `d0bb6a8464fcacfbed180a79e5082261060d30691c0a835492f43b3681b72870`。
中文操作方式與驗證記錄：`<project-root>/outputs/ValheimCli-0.2.0-操作方式.md`、`<project-root>/outputs/ValheimCli-0.2.0-驗證結果.json`。project-root 是 2026-10-08/new-chat-4 工作目錄。
