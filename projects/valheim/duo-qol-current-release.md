---
title: "Valheim Duo QoL v61：模組更新、負重設定遷移與拾取網格設定"
scope: projects/valheim
status: verified-packaging-runtime-unverified
updated: 2026-10-18
---

截至 2026-10-18，本次核對的正式交付為 v61，取代此筆先前的 v58 狀態。續作先讀目前 repository、lock 與交付，不把歷史版本當成當前版本。

- 專案 `<user-documents>/valheim-duo-qol`，公開 repo `swear01/valheim-duo-qol`。v61 檔名 `Valheim-Duo-QoL-v61.r2z`，profile `Valheim-Duo-QoL-20261018-v61`；51 個線上套件、42 份公開設定、43 份私人設定。
- 雙人交付 `<project-root>/deliveries/2026-10-18/Valheim-Duo-QoL-v61.r2z`，與正式私人重建逐位元組相同，權限 0600。公開／私人同名，不能拿公開下載代替私人交付。
- 核對 51 個線上套件與兩個 Nexus 模組後，只更新 ExtraSlots 1.2.20 → 1.2.21、SkilledCarryWeight 1.5.0 → 1.6.1；其餘 49 個線上套件及內附 DLL 保留。這是當次查核結果，不是之後仍為最新版的保證。
- ExtraSlots 1.2.21 修正舊備份恢復出重複物品：只保留最新快照，每次新備份取代舊備份與未解決記錄。既有欄位數、食物格、進度與快捷鍵保留。不得推定它已修復另行回報的種植重疊／扣料問題。
- SkilledCarryWeight 1.6.1 修正力量腰帶加成，新增基本負重設定，移除 Quick Cart 功能。既有快速拉車鍵為 None，因此不補裝替代模組；基本負重固定 300、只有 Run 啟用、Coefficient 4、Power 1，推車減重關閉。
- 負重設定須實際遷移，不能只改版本：1.5.0 使用零寬字元章節／Enabled 鍵；1.6.1 改用 Jotunn `BindConfigInOrder`，章節是 `1 - Global`、`2 - Cart Mass`，技能也要編號。隱藏的 None 技能仍占第 3 個章節，因此 Swords 是 `4 - Swords`、Run 是 `20 - Run`、Ride 是 `27 - Ride`；Enabled 鍵改為普通字串。只刪零寬字元會讓覆寫失效。依據新版 DLL、Jotunn 2.30.2 的章節排序 helper，以及本機遊戲參考組件的 Skills enum／Enum.GetValues；尚未遊戲實測。
- BepInExPack 5.4.2351 與 Jotunn 2.30.2 在 v60 已安裝，正好符合 SkilledCarryWeight 1.6.1 的需求；不在本次 diff 不代表版本較舊。審查若宣稱缺少依賴，先核對完整 lock／export，再決定是否修改。
- Torch Light Mod 仍為 Nexus 官方未修改的 1.0.2 DLL（mod 11、file 25236）；作者 GitHub 仍宣告 1.0.1，不能以舊 source 重建冒充 1.0.2。BepInPlugin 是 1.0.2，AssemblyVersion 仍是 1.0.0.0。Badgers 維持 0.6.0，只在私人包。

目前選擇與設定：

- ExtraSlots 的 3 個食物格已開啟；保留 1 列額外背包、3 個一般快捷格、解鎖進度與備份。AzuAutoStore 的 `Player Ignore Quick Slots = On` 保留；食物格實際不被自動收納與續吃仍未在遊戲內驗證。不要在特殊格仍有物品時移除或縮減 ExtraSlots。
- Hunger Pangs 1.3.3：先手動選好食物並保留庫存，剩餘 25% 時續吃同種食物。工作台／船上暫停食物計時與四種自動喝藥皆關閉；死亡保留已吃食物效果與剩餘時間、TimeControl 連動皆開啟。包內未裝 TimeControl，連動目前不改變時間。AutoNosh／AutoEat TT 沒有安裝。
- 藥水專用快捷欄暫時放棄，不要自行加入 PackPanel 或替換 ExtraSlots。使用者決定維持 SlashBack 1.0.2 加內附 SlashBack Return Fix 1.0.0：取消冷卻與死亡期限，保留 5 秒站立倒數與移動取消；地圖傳送維持關閉，不替換 Waystones。
- Badgers Straight Structures 0.6.0 只內附私人包，保留作者 `Warp Amount = 0.004`，會一起調整石材、木牆、樑柱與門的視覺扭曲，沒有石材限定設定。作者禁止重新上傳，DLL／私人 ZIP 不進公開 repo、模組站或 Actions。
- PerfectPlacement 自 v53 移除，F1 精確建築不再使用；PlanBuild 自 v47 移除。保留 Infinity Hammer、World Edit Commands、MovingDay。ComfortTweaks 3.3.11 判定範圍 50、清單開關 J；Home 會衝突 MovingDay 高度重設。
- AzuCraftyBoxes 取料半徑 50、單箱切換 RightShift；PlantEasily 範圍修飾鍵 RightShift、採收 LeftShift。Infinity Hammer 左右排列 RightAlt＋滾輪，前後排列 RightShift＋滾輪，縮放 None；對著箱子按 RightShift 可能一起切換取料。ValheimTerrainTools Removal 與 Brush 上限 10，Remove builds 沒有材料返還或復原。
- 營火沿用 v32 RGB 解法：`FirePitIntensityLow = 1.5`、`FirePitIntensityHigh = 2`、`FirePitColorLow = 35211AFF`、`FirePitColorHigh = 402014FF`。舊 1.0.1 的隔離四營火實測不能當成 1.0.2 的畫面驗證。

- v59 起 Server devcommands 設 `Automatic debug mode = false`、`Automatic fly mode = false`、`Automatic no cost mode = false`、`Disable debug mode keys = true`。原版 Z／B 是除錯模式寫死的飛行／免材料鍵，不能真設成 None，透過關閉自動除錯及除錯鍵防誤按；手動指令保留，更新封包不會重設目前已開啟的遊戲狀態。
- v60 加入 Super Configurable Pickup Radius 1.1.2：`Insanely Configurable / Enable Pickup Range = true`、`Pickup Range = 10`，需完全重啟遊戲才套用 Player.Awake 的拾取半徑。
- 使用者明確選擇關閉 PlantEasily 世界方向對齊、保留整排種植：`Grid / GloballyAlignGridDirections = false`；ModActive／SnapActive、5×5 陣列、吸附、HarvestRadius 5、自動補種與材料／耐力／耐久消耗設定保留。

私人交付固定執行 `python3 -m scripts.build_private_package`。必須沿用本機 `<user-home>/.config/valheim-duo-qol/private-admins.json` 的兩位管理員，以及同目錄 BadgersStraightStructures.dll；任一缺失／格式或原始 DLL 雜湊不符就停止，不能以公開包代替。私人名單、permissions.yaml、私人來源內容與其 hash 都不寫共享記憶或公開 Git。私人包會加入 Badgers、權限檔並開啟 Automatic devcommands；GitHub／共享記憶不提供私人名單或 Badgers 備份。

主分支只留 `history/v61/`，歷史原始 archive／lock 留在對應 tags，不改寫已發布 tags。兩人退出遊戲、備份角色／世界後，匯入同一份私人包為新 Gale profile，勾選 Import all files 才會複製內附 DLL。

v61 公開 archive SHA256：`c0fd15dfad49389200a21bff4e751db7a2df0e6608bd8970d114070fb06bd2ea`。私人交付雜湊留在本機驗證紀錄，這裡不記錄私人來源或名單。

模組包 PR #30 已合併；head 與 v61 tag 為 `ceb28a9880080fabda7cad85388540f6d42ea9b1`，合併 main 為 `9f009b1f49c157b6a2342ab2de9639e154eb71d5`。17 項 Python 測試、27 項 SlashBack 檢查及相同 DLL 重建、61 版 archive／lock／tag／profile、依賴與設定、公開／私人重建、v60→v61 私人 ZIP 差異、隱私檢查與 0600 均通過；私人權限與 Badgers 逐位元組不變。三項同 head CI 通過，Codex code／security review 在同 head 均 completed 且無 findings，依 OR 規則採第一個通過者。Swear 為模型／OCR 服務失敗、Cursor 為額度用盡，均不是程式檢查通過；Gemini 依賴誤報已以完整 lock 舉證。本機已同步 main，任務分支／worktree 已清理。

Gale GUI 匯入、新版 ExtraSlots 備份恢復、負重／腰帶／其他模組互動、Torch 火光、食物格續吃／自動收納排除、死亡食物保留、Unity／Harmony 與雙人／長時間行為皆未在本次實測；沒有啟動遊戲或修改存檔。種植重疊／扣料異常仍未確認修復。證據為 `<project-root>/lock.json`、`history/versions.json`、`research/mod-updates-v61-20261018/verification.json`、`pre-merge-verification.json`、`merged-verification.json` 與模組包 PR #30。記憶同步及 QMD 索引要另做遠端與搜尋讀回，不由模組包檢查推定。
