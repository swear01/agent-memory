---
title: "Valheim Duo QoL v58：ExtraSlots 備份修正與 Torch 官方 DLL"
scope: projects/valheim
status: verified-packaging-runtime-unverified
updated: 2026-10-16
---

截至 2026-10-16，本次核對的正式交付為 v58，取代此筆先前的 v52 狀態。續作先讀目前 repository、lock 與交付，不把歷史版本當成當前版本。

- 工作目錄為 `<user-documents>/valheim-duo-qol`，公開 repo 為 `swear01/valheim-duo-qol`。v58 檔名 `Valheim-Duo-QoL-v58.r2z`，profile `Valheim-Duo-QoL-20261016-v58`；50 個線上套件、41 份公開設定、42 份私人設定。
- 雙人交付為 `<project-root>/deliveries/2026-10-16/Valheim-Duo-QoL-v58.r2z`，與 `dist/private/` 的正式私人重建逐位元組相同，權限均為 0600。公開／私人同名，要依來源與雜湊區分。
- ExtraSlots 1.2.19 → 1.2.20（Hexium）：修正 Valheim 1.0.20 後額外欄位備份無法恢復。DLL 靜態比對確認序列化改讀目前遊戲的 item data version，移除寫死 109 的上限並保留完整資料邊界檢查；不是遊戲內恢復成功的證據。
- Torch Light Mod 1.0.1 → 1.0.2：使用 aedenthorn 在 Nexus 發布的未修改官方 DLL，mod 11、file 25236。GitHub 原始碼仍宣告 1.0.1，不能冒充可重建的 1.0.2 來源。Nexus 允許署名重新上傳、禁止把資產用於販售的模組／檔案，包內授權與 THIRD_PARTY.md 已更新。BepInPlugin 版本是 1.0.2，AssemblyVersion 仍是 1.0.0.0；判斷插件版本須讀前者。
- 所有設定值與其餘 49 個線上套件保留，只改兩份 CFG 的說明標頭、manifest 與 Torch DLL／授權。ExtraSlots 與 Torch 新舊 DLL 分別有 89、83 組相同設定鍵。

目前選擇與設定：

- ExtraSlots 的 3 個食物格已開啟；保留 1 列額外背包、3 個一般快捷格、解鎖進度與備份。AzuAutoStore 的 `Player Ignore Quick Slots = On` 保留；食物格實際不被自動收納與續吃仍未在遊戲內驗證。不要在特殊格仍有物品時移除或縮減 ExtraSlots。
- Hunger Pangs 1.3.3：先手動選好食物並保留庫存，剩餘 25% 時續吃同種食物。工作台／船上暫停食物計時與四種自動喝藥皆關閉；死亡保留已吃食物效果與剩餘時間、TimeControl 連動皆開啟。包內未裝 TimeControl，連動目前不改變時間。AutoNosh／AutoEat TT 沒有安裝。
- 藥水專用快捷欄暫時放棄，不要自行加入 PackPanel 或替換 ExtraSlots。使用者決定維持 SlashBack 1.0.2 加內附 SlashBack Return Fix 1.0.0：取消冷卻與死亡期限，保留 5 秒站立倒數與移動取消；地圖傳送維持關閉，不替換 Waystones。
- Badgers Straight Structures 0.6.0 只內附私人包，保留作者 `Warp Amount = 0.004`，會一起調整石材、木牆、樑柱與門的視覺扭曲，沒有石材限定設定。作者禁止重新上傳，DLL／私人 ZIP 不進公開 repo、模組站或 Actions。
- PerfectPlacement 自 v53 移除，F1 精確建築不再使用；PlanBuild 自 v47 移除。保留 Infinity Hammer、World Edit Commands、MovingDay。ComfortTweaks 3.3.11 判定範圍 50、清單開關 J；Home 會衝突 MovingDay 高度重設。
- AzuCraftyBoxes 取料半徑 50、單箱切換 RightShift；PlantEasily 範圍修飾鍵 RightShift、採收 LeftShift。Infinity Hammer 左右排列 RightAlt＋滾輪，前後排列 RightShift＋滾輪，縮放 None；對著箱子按 RightShift 可能一起切換取料。ValheimTerrainTools Removal 與 Brush 上限 10，Remove builds 沒有材料返還或復原。
- 營火沿用 v32 RGB 解法：`FirePitIntensityLow = 1.5`、`FirePitIntensityHigh = 2`、`FirePitColorLow = 35211AFF`、`FirePitColorHigh = 402014FF`。舊 1.0.1 的隔離四營火實測不能當成 1.0.2 的畫面驗證。

私人交付固定執行 `python3 -m scripts.build_private_package`。必須沿用本機 `<user-home>/.config/valheim-duo-qol/private-admins.json` 的兩位管理員，以及同目錄 BadgersStraightStructures.dll；任一缺失／格式或原始 DLL 雜湊不符就停止，不能以公開包代替。私人名單、permissions.yaml、私人來源內容與其 hash 都不寫共享記憶或公開 Git。私人包會加入 Badgers、權限檔並開啟 Automatic devcommands；GitHub／共享記憶不提供私人名單或 Badgers 備份。

主分支只留 `history/v58/`，歷史原始 archive／lock 留在對應 tags，不改寫已發布 tags。兩人退出遊戲、備份角色／世界後，匯入同一份私人包為新 Gale profile，勾選 Import all files 才會複製內附 DLL。

v58 公開 SHA256：`bae9591b971b5b1e4e03c36a3ffe0111287433d319ebf504c793b316cf76910b`；私人包 SHA256：`b40e5843b92bd6ed913d8081386f5d6a23e85e58f0193e53b007c718456870e3`。

模組包 PR #27 已合併；head 與 v58 tag 為 `f9aa05e621adc2a9044521f6d13b83cfd6ce87ad`，合併 main 為 `65bdcf40e94aa088a6550f519f0381a75e1125ea`。17 項 Python 測試、27 項 SlashBack 檢查及相同 DLL 重建、58 版 archive／lock／tag／profile、依賴與設定、公開確定性重建、實際 v57→v58 私人 ZIP 差異、隱私檢查與 0600 均通過。三項同 head CI 通過，Gemini Code Assist 對該 head 給出無意見的完整審查；依 OR 規則採第一個通過者，沒有另外手動觸發其他 reviewer。模組包任務分支／worktree 已清理。

Gale GUI 匯入、新版 ExtraSlots 備份恢復、Torch 火光、食物格續吃／自動收納排除、死亡食物保留、Unity／Harmony 與雙人／長時間行為皆未在本次實測；沒有啟動遊戲或修改存檔。證據為 `<project-root>/lock.json`、`history/versions.json`、`research/mod-updates-v58-20261016/verification.json` 與模組包 GitHub PR #27。記憶同步及 QMD 索引要另做遠端與搜尋讀回，不由這些模組包檢查推定。
