---
title: "Valheim Duo QoL V32：營火 RGB 調暗歷史證據"
scope: projects/valheim
status: verified-packaging-runtime-partial
updated: 2026-10-13
---

# V32 歷史交付與營火修正

2026-10-13 續作基底已更新至 v41，見 [目前交付與私人雙人設定](duo-qol-current-release.md)。以下只保存 V32 當時的營火修正證據，不代表目前最新版本。

2026-10-12 當時的正式交付為 `Valheim-Duo-QoL-v32.r2z`，profile 名稱 `Valheim-Duo-QoL-20261012-v32`。檔案位於當時模組包工作目錄的 `outputs/`。

- 完整包：`Valheim-Duo-QoL-v32.r2z`，22,595 bytes，SHA-256 `4a176dfc6887ab6a440cabe8c471d417e475fea548196235e6540f81d48f61c3`。
- 版本／來源／雜湊：`Valheim-Duo-QoL-lock-v32.json`。
- 設定備份：`Valheim-Duo-QoL-config-v32.zip`；指南：`Valheim-雙人模組包-安裝指南-v32.html`。
- 已有 V31 時只需 `Valheim-營火RGB設定增補-v32.zip`（380 bytes，只有一份 CFG）；若自行改過該 CFG，只改下列四項。兩人正常退出遊戲後套用，不要為了檢查而自行重開目前遊戲。

V32 以 V31 為基底。45 個線上套件的版本／來源與其餘 35 份設定逐位元組保留，共 36 份設定。完整包只有 TorchMod 是內附 DLL；完整匯入 Gale 時須勾選 Import all files。未加入 GraphicsOverdrive、LightTweaks 或額外光照修補 DLL。

AAA Crafting Max Fix 已發布為 `swear01-AAA_Crafting_Max_Fix-1.0.0`，由 Thunderstore 下載，不內附第二份 DLL。此修補對應 Hexium 的 AAA Crafting 2.1.11 與 AzuCraftyBoxes 1.8.27；上游版本或官方修復狀態有變時須重新驗證。官方確認修好同一問題後，驗證並移除本修補依賴。

# TorchMod 真正原因與採用設定

Valheim 1.0.17、Torch Light Mod 1.0.1 的隔離四營火測試確認：TorchMod 的 Fireplace.Start postfix 在尚未啟用的營火燈光初始化前寫入 LightFlicker.m_baseIntensity。之後 LightFlicker.Awake 從 Light.intensity 重新讀值，把快取覆寫。例如設定 0，Start 後快取 0，Awake 後又成 2；有效能模組 VPO 與沒有 VPO 都會重現。因此，單純把 Intensity 降低不能當成已生效。

採用現有 TorchMod 的 Light.color RGB 路徑調暗周圍照明，並保留觀測到的低／高光源基準。正式 CFG 為 `BepInEx/config/aedenthorn.TorchMod.cfg`：

```ini
[Fire Pits]
FirePitIntensityLow = 1.5
FirePitIntensityHigh = 2
FirePitColorLow = 35211AFF
FirePitColorHigh = 402014FF
```

TorchMod DLL 未修改，SHA-256 `c79b8106e7aff0d646b9be75a0e731ffd856a35f47a73471078bd5c419f9e77a`。V32 只改上述四值、CFG 的版本註記與 export 的 profile 名稱。

BetterFog 1.0.3 保持作者原生設定：Fog.Density=0.5、Fog.MistOpacity=0.2、Smoke.Opacity=0.5。不要沿用先前的煙霧 90% 候選設定。

# 商店替代品調查與使用者決定

使用者希望多個火焰重疊時仍看得清楚物品，且優先使用 Thunderstore／Hexium 的依賴下載。最終選擇保留 TorchMod，採用上述 RGB 方法。

- LightTweaks 0.1.6 可調燈光強度／範圍／閃爍，但把營火可蓋在木地板的限制直接解除，沒有 config 開關，不能聲稱只靠設定可關掉。
- GraphicsOverdrive 1.1.4 有 Bloom 強度／門檻／半徑與整體曝光控制；一般渲染沒有營火專用亮度設定。PointLightStrength 只在實驗性光線追蹤路徑。全新安裝通常選 Balanced，會同時改陰影、燈光距離、煙霧大小；其 FogDensityScale 與 BetterFog 相乘。未安裝或進遊戲測試，不要宣稱能取代 TorchMod。
- 原生 Bloom 關閉能減少光暈，但火焰核心仍亮；先前隔離測試的原設定已關閉 Bloom。BloomSettings 能保留較弱光暈，不能據此宣稱降低營火實際照明或火焰材質亮度。

# 驗證範圍與證據

V32 已完成獨立讀回：ZIP CRC／安全路徑、精確 V31 差異、45 個套件 pins／來源／依賴、TorchMod DLL 雜湊、三種 ZIP 設定一致、lock 設定一致、BetterFog 原生值、指南本機連結與記錄的輸出雜湊。沒有修改正式 Gale profile 或重開目前遊戲。

已重新跑過先前隔離實測的證據斷言：VPO 有／無時的晴天高光源 RGB，以及無 VPO 時的雨天低光源 RGB 與晴雨切換均有證據。原因與對照報告為 `outputs/Valheim-營火亮度-原因與設定-20261012.html`；證據位於 `work/torch-live/component-probe-20261012/`，含 `results.json`、樣本、生命周期紀錄與 `check-evidence.py`。

尚未完成 V32 整包 Gale GUI 匯入、雙人、長時間測試，以及有 VPO 時的雨天低光源組合。RGB 只改 Light.color，火焰粒子／材質核心仍可能偏亮；不得把隔離四營火證據擴大成完整模組包或多人驗證。
