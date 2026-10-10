---
title: PlantEasily planting grid adjustment without Right Ctrl
scope: projects/valheim-qol
status: source-verified-runtime-unverified
updated: 2026-10-10
---

# 沒有右 Ctrl 的種植操作

PlantEasily 2.3.0 的 Controls / KeyboardModifierKey 預設 RightControl；本工作區 outputs/v18/advize.PlantEasily.cfg 也保留此值，General Rows / Columns 為 5 / 5。不能以此認定同學當前 Gale profile 檔案完全相同。

沒有右 Ctrl 可由遊戲內 Configuration Manager（此包使用 F10 / Mods settings）→ PlantEasily → Controls → KeyboardModifierKey 改 LeftControl。按鍵設定變更有 SettingChanged → KeybindsChanged 處理。若無法使用遊戲內設定，退出遊戲後編輯目前 Gale profile 的 BepInEx/config/advize.PlantEasily.cfg，將 `[Controls]` 中 `KeyboardModifierKey = RightControl` 改為 `KeyboardModifierKey = LeftControl`，再啟動遊戲。

必須裝備耕耘器並選擇支援的作物：按住設定的修飾鍵，→ 增加欄、← 減少欄、↑ 增加列、↓ 減少列。IncreaseXKey / DecreaseXKey / IncreaseYKey / DecreaseYKey 亦可自訂。也可直接設定 General / Rows 與 Columns，例如 3 / 3 為 3×3 種植範圍。批次採收是另外的 KeyboardHarvestModifierKey，預設 LeftShift，配合使用鍵；不要把它誤改成調整種植大小的設定。

作者說明 https://thunderstore.io/c/valheim/p/Advize/PlantEasily/ 已核對：所有按鍵可設定、只在耕耘器與適用作物選定時生效、支援遊戲內 Configuration Manager。2.3.0 預設尊重客戶端設定（耐力與耐久除外）；若 server 另外強制同步則另查。不需要 OP 才能一般本機改鍵。

此輪僅核對作者文件與本機 ModConfig.cs / DLL 反編譯，未變更同學電腦或目前模組包，也未實際種植。
