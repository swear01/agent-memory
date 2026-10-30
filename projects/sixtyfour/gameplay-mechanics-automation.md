---
title: Sixty Four 後期數值公式、泯滅保護與自動化機制
scope: project
project: sixtyfour
status: active
updated: 2026-10-30
---

# 後期核心資源與數值機制

Windows 遊戲 1.2.1 經原始程式碼（`game.js`、`stuff.js`）與實機驗證的數值公式與運作邏輯：

## 1. 泯滅反應（Annihilation / Crusade）與地獄寶石保護

- 當深度達 18,000m+ 解鎖天體泡沫（Celestial Foam，`resources[6]`）後，`updateResourceInteractions()` 觸發泯滅反應：
  `hellGems > vaults.size * 1024` 時，未受保護的地獄寶石會與天體泡沫以 `hellgemChunk` 為單位發生泯滅。
- 基礎 `hellgemChunk = 64`。升級 `strange1`（Hollow Rock Research Site）後，泯滅單元提升為 `512`。
- 每次泯滅將地獄寶石轉化為 4 倍的 Beta-Pylene（`hellToDestroy * 4`），並向 `this.annihilationMachines` 發送 tap 訊號。
- **Hell Vault（`vault`）保護機制**：
  每個 Hell Vault 隔絕 1,024 顆 Hell Gem 不受泯滅。建造 2 座 Hell Vault 即可安全保留 2,048 顆 Hell Gem，滿足後續建造湮滅機（Annihilator）的門檻。

## 2. 虛空石研究站（Hollow Site）與硬度衰減

- `strange1`（位於坐標 `(-100, -100)`）定期在半徑 8 格內生成 Hollow 實體（上限 8 個）。
- Hollow 具有 `integrity = 1`，點擊損耗公式為 `integrity -= 1 / hollowHardness`。
- 初始 `hollowHardness = 64`（需點擊 64 次破裂）。破裂時掉落 1 顆 Hollow Stone（`resources[7]`），並觸發硬度減半：
  `hollowHardness = Math.max(4, hollowHardness / 2)`。
- 歷經 64 → 32 → 16 → 8 → 4 後，後續生成的 Hollow 僅需 4 次點擊即可破碎。

## 3. 轉化器（Converters）與後期黑色／紫色資源飛輪機制

- **深層地層資源斷層**：深度達 10,000m～30,000m+ 後，地層方塊組成僅含天體泡沫（約 75%～90%）與 Beta-Pylene（約 10%～25%），地層已完全不再天然生成黑色（Charonite）與紫色（Qanetite）方塊。
- **黑色方塊（Charonite）後期唯一製造源**：
  社群與實機驗證確認，後期黑色方塊完全依賴 **Beta-Pylene 氧化器（`converter41`）** 工業化轉化。每座運轉一次消耗 8,192 Beta-Pylene，產出 **16,384 Charonite**。
- **紫色方塊（Qanetite）供應鏈**：
  由 **Charonite 沉澱槽（`converter13`）** 消耗 4,096 Charonite 產出 **256 Qanetite**。若黑色方塊匱乏，紫色方塊供應將徹底停擺，導致地獄寶石注入器（Injector）斷油，方塊失去地獄寶石，破壞器（`destabilizer2a`）失去 625 倍破碎加成，造成主坑挖掘停滯。
- **冷啟動拆解法（Scrap & Rebuild）**：
  若因斷產導致黑色與紫色雙雙見底，可拆除外圍冗餘的 Elmerine 槽（`converter32`）或次要共振器，取得高額黑色與紫色方塊退款，立即注入 `converter13` 啟動紫色方塊，隨即擴建 `converter41` 矩陣，引爆「Beta-Pylene → 16,384 Charonite → 256 Qanetite → Injector 注入 Hell Gem → 625x 秒碎方塊 → 巨量 Beta-Pylene」之自持飛輪。

## 4. 湮滅機（Annihilator）與虛空（Void）生成

- 建造門檻：Charonite 8.38M, Elmerine 524k, Qanetite 1.04M, Hell Gem 2048, Chromalit 16384, Hollow Stone 32。
- 運作機制：以 1 顆 Hollow Stone 為燃料（`fuel = [0,0,0,0,0,0,0,1]`），每次環境發生泯滅反應時，`annihilationMachines` 自動呼叫 `m.tap()`。
- 每次 tap 消耗 0.03125 fill（單顆 Hollow Stone 可支撐 32 次 tap），並產生 1 單位 Void（`resources[8]`）。

## 5. 虛空雕塑（Void Sculpture）與反轉深淵（The Inverse Chasm）

- 虛空雕塑（`voidsculpture`）消耗 1,024 Void，能吸收泯滅反應引發的畫面震動（`createHollowEvent`），並啟用暗面懸浮收集。
- 虛空雕塑建造後可進一步解鎖反轉深淵（`chasm`）與終局維度躍遷。
