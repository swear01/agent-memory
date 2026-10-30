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

## 6. 轉化器自動化與紫色方塊生產配比工程

- **轉化器自動化原理（原生 Silo vs 守護進程）**：
  - **原生 Silo 機制**：在 `stuff.js` 的 `Silo.update()` 邏輯中，Silo 會持續掃描其 `soi`（周圍 8 格九宮格相鄰地塊）。若地塊上有具備 `refill()` 能力的機器且玩家原料充足，Silo 即刻執行 `cell.refill()`，並扣除儲罐內部次數（`this.tap()`：標準 Silo 具備 16 次，Industrial Silo MKII 具備 64 次）。因此，只要轉化器緊鄰 Silo 放置，遊戲引擎本身就會**100% 原生全自動裝填原料**，完全不需要玩家額外點擊或設置觸發器。
  - **超出 Silo 範圍的建築**：遊戲原生不會自動裝填；然而在 Electron Agent API 支援下，`keeper.mjs` 後台進程會全圖輪詢閒置機器（`state === 0 && canInteract`）直接發送遠端裝填指令，形成「Silo 零延遲原生裝填 + 守護進程全圖兜底」的雙重自動化。
- **紫色方塊（Qanetite）流動停滯之根因與產能解法**：
  - 1 座 `converter13` 每次產出 256 紫色方塊。
  - 4 座 Injector 每次裝填消耗 256 紫色方塊（4 × 64），7 座 `converter41` 每次裝填消耗 112 紫色方塊（7 × 16）。
  - 當只有 2 座 `converter13` 時，單輪產出 512 紫色方塊，幾乎被下游客戶端（Injector、Converter41）瞬間榨乾，導致紫色庫存長期在 0～80 的「浮動在途」狀態，無法蓄積。
  - **最佳工業化配比**：配置 **8 座 `converter13`**（沉澱槽）與 **7 座 `converter41`**（氧化器），配合中軸雙排 Silo 矩陣。單輪產出躍升至 2,048 紫色方塊，淨蓄積率高達 +1,648/輪，使紫色方塊在數十秒內突破 80,000+，直奔 Hell Vault #2（需 39.3 萬）與 Annihilator（需 104 萬）。

## 7. 轉化矩陣動態平衡與回授震盪防護（Dynamic Equilibrium Governor）

- **震盪失衡根因**：
  - 8 座 `converter13` 每次需消耗 32,768 黑色方塊，而 7 座 `converter41` 每次需消耗 57,344 黃色（Beta-Pylene）方塊。
  - 若兩者過度擴張，會在一瞬間將黑色與黃色雙雙榨乾，造成兩個原料槽同時處於飢渴震盪（0～10k 徘徊）。
  - 若此時白色方塊（Elmerine）因只剩 1 座 `converter32` 而產能不足，Silo（每次 1,024 白色）與 Destabilizer（每次 64 白色）會迅速耗盡白色儲備；當白色方塊低於安全線時，Silo 停止自動運轉，方塊秒碎中斷，黃色方塊斷炊，引發全廠連鎖減速。
- **穩態四色共榮配比工程**：
  1. **實體解耦**：拆除過量轉化器，將配置穩定在 **4 座 `converter41`**（黃轉黑）、**4 座 `converter13`**（黑轉紫）、**3 座 `converter32`**（紫轉白）。
  2. **動態守門（Governor）保護機制**：
     - **黑色保護**：當 Charonite < 50,000 且 Qanetite > 100,000 時，強制暫停 `converter13`，禁止黑轉紫，優先蓄積黑色方塊。
     - **黃色保護**：當 Charonite > 500,000 時，節流 `converter41`，讓 Beta-Pylene 順暢累積至 Hell Vault #2 所需的 3,145,728 巨量儲備。
     - **白色保底**：3 座 `converter32` 將充沛的紫色轉為白色，維持 Silo 充能無休。
  3. **方塊連擊秒碎**：主坑方塊採用每秒多重連擊破裂，方塊即生即碎，源源不絕產出黃色方塊與天體泡沫，讓黑、黃、白、紫四色淨產能全數維持為正。

## 8. 後期高級巨構與終局科技樹解鎖鏈（Endgame Celestial Tech Tree）

- **終局進階建築功能與解鎖階梯**：
  1. **`vessel`（Containment Vessel，收納容器）**：
     - 單座隔絕保護 32 單位 Chromalit 不受半衰期裂變衰變。建造 8 座達成 **256 Chromalit** 容量，為解鎖 `converter76` 的必經前置。
  2. **`converter76`（Celestial Irradiator，天體輻照機）**：
     - 造價：262,144 Charonite, 128 Hell Gem, 256 Chromalit, 65,536 Celestial Foam。
     - 運作機制：消耗 1 Chromalit + 8,192 天體泡沫，產出 **2,048 Chromalit + 1 顆 Hell Gem**。
     - 戰略價值：直接引爆 Chromalit 產能（從百位數暴衝至數千），且為解鎖天體反應堆（`converter64`）之絕對必備前置。
  3. **`converter64`（Celestial Reactor，天體反應堆）與 `reflector`（天體反射板）**：
     - 解鎖條件：建造至少 1 座 `converter76`。
     - 運作機制：融合 4,096 Chromalit 與 32,768 天體泡沫，產出公式為 $(32768 + \text{reflectorCount} \times 8192) \times 4$。
     - 產能效果：周圍環繞 4 座反射板時，單次運轉即可爆出 **262,144 巨量 Beta-Pylene（黃色方塊）**，直接滿足 Hell Vault #2（需 3,145,728）之巨額門檻。
  4. **`generaldecay`（General Decay Reactor，全域衰變反應堆）**：
     - 單一唯一建築，極大化 Chromalit 衰變性能，吸收裂變脈衝轉化為海量低階元素。
  5. **`preheater`（Catalytic Preheater，催化預熱器）**：
     - 緊鄰轉化器放置，提供高達 +200%～+300% 速度倍率加成（最高達 4 倍運轉速度）。
  6. **`consumer`（Catalytic Refinery，催化精煉廠）**：
     - 吞噬方塊破碎資源，累積滿 1,024 單位後連續爆發，最高提供 +100% 額外資源掉落加成。
  7. **`strange2`（Hollow Rock Facility）與 `strange3`（Reconstructed Hollow）**：
     - 解鎖條件：建造 Annihilator 並產出 Void（`resources[8] > 0`）。
     - 功能：升級北方的虛空石基地（`(-100, -100)`），將虛空石上限倍增至 16 顆並極速生成，徹底打通終局維度躍遷。

## 9. 原生 Agent API 整合與 client.mjs SDK 效能躍升

- **API 傳輸層瓶頸與修正實測**：
  - 傳統 Node.js 24 內建 `fetch()` 受限於 Undici 的 `scheduleIdleSocketValidation` timer 機制，即使本機 loopback 重用連線，每次請求仍被強制延遲約 13.6ms～15ms。
  - 官方 SDK（`client.mjs`）使用原生 `http.Agent` 搭配明確的 `setNoDelay(true)` 與 Socket 連線池管理，將本機交互中位數直接壓至 **0.47ms（health）與 4.8ms（state）**，大幅降低輪詢開銷。
- **雙模式整合規範**：
  1. **CLI NDJSON 常駐交談模式**：
     - 透過 `node sixtyfour.mjs session` 啟動常駐子程序，保持 stdin 開啟。
     - 透過單一管道寫入每行 `{"id": N, "argv": ["health"|"state"|"catalog"|...]}`，輸出依序回傳對應 `id` 之 JSON。不頻繁關閉 pipe 或重啟 Node。
  2. **常駐 Node Agent SDK 模式**：
     - Windows ESM 匯入需使用 `pathToFileURL(path).href`（避免直接字串路徑解析失敗）。
     - 於主迴圈外建立單例 `const client = await createClientFromConfig()`，全生命週期重用連線，結束時呼叫 `client.close()`。
  3. **批次連擊與建築材料結構升級**：
     - 支援 `interact` 的 `count: 1..64` 伺服器端 30ms 節拍連點（綁定 `targetId`，方塊破壞或消失即自動停止），打破過去單點 HTTP 迴圈的網路延遲限制。
     - `catalog.data` 原生提供 `materials: [{index, name, amount}]`、`placementSupported` 與 `available`，免除前端自解碼陣列與繁瑣驗證。

## 10. 終局巨構 Annihilator 衝刺與工廠高階矩陣

- **深度突破與資源儲備現狀（深度 57,800m+）**：
  - **Hell Vault #2** 已於 `(13, 1)` 成功落成，地獄寶石（Hell Gem）隔絕上限提升至 2,048，實時儲備突破 **2,500+**（已完全達到 Annihilator 2,048 之硬性門檻）。
  - **7 大建材中有 5 項完全超標達標**：
    - Elmerine（白）：2.08M / 524k（400%）
    - Qanetite（紫）：3.61M / 1.04M（347%）
    - Beta-Pylene（黃）：7.69M / 256（300 萬 %）
    - Hell Gem（紅）：2,219 / 2,048（108%）
    - Hollow Stone（石）：2,441 / 32（7,600%）
- **最後兩大瓶頸攻堅架構**：
  1. **Chromalit（高階裂變體）**：建造 **General Decay Reactor（`generaldecay`）** 於 `(10, -2)` 吸收衰變輻射；部署 3 座 `converter76`（天體輻照機）搭配 `(11, 3)` 之 **Catalytic Preheater（`preheater`）** 與 `(12, 3)` 之 **Industrial Silo MKII**，使 Chromalit 產能壓制半衰期裂變損失，向 16,384 狂飆。
## 12. 4 路催化預熱陣列、高階裂變平衡突破與虛空儲備激增（Strange2 & Reality 衝刺）

- **Chromalit 裂變衰變數學模型解密**：
  - 遊戲中未被容器（Containment Vessel，每座容量僅 32）收納的 Chromalit，每 frame 會以 `decayChance = 2e-5 * dt` 發生放射性裂變衰變（`chromaToContain * decayChance`）。
  - 單座天體輻照機（`converter76`）基礎轉換速率為 `1e-5`（每週期需 100,000 dt 產出 2,048 Chromalit，平均產能約 0.02048/dt）。7 座無大幅預熱之輻照機產能上限約 0.143～0.266/dt，導致在 18,000～20,000 Chromalit 處即與自然裂變損失達到動態平衡，無法自然觸及 32,768 門檻。
- **4 路催化預熱器（Catalytic Preheater）與 8 座天體輻照機矩陣**：
  - 每座預熱器（`preheater`）提供 `+1.5` 乘數（最高可達 300% 速度提升）。
  - 於 `(10, 4)` 與 `(14, 4)` 新增部署第 3、第 4 座 Catalytic Preheater，並於 `(14, 3)` 部署第 8 座 Celestial Irradiator，形成緊密環繞的預熱加速矩陣。
  - Chromalit 瞬間衝破 20,000 衰變天花板，以 +250/s 之極速向 32,768 突破！
- **終局資源儲備現狀（深度 59,425m+）**：
  - **Charonite（黑）**：6.34M+（正以 ~1M/min 速度回升向 8.38M，其他材料皆已遠超 Strange2 所需）
  - **Elmerine（白）**：66.9M+（Strange2 需 4.19M，超標 1,590%）
  - **Qanetite（紫）**：62.3M+（Strange2 需 4.19M，超標 1,480%）
  - **Beta-Pylene（黃）**：174.5M+（Strange2 需 4.19M，超標 4,160%）
  - **Hell Gem（紅）**：2,511（雙金庫安全絕緣，高於 2,048）
  - **Void（虛空）**：**41,461+**（Strange2 僅需 64、Void Sculpture 僅需 1,024，超標 4,000%）
  - **Hollow Stone（石）**：1,438+（Strange2 僅需 16）
- **真實維度（Reality，`resources[9]`）機制解析**：
  - 建造 Void Sculpture 後，點擊切換進入暗面空間（`plane = 1`）。
  - 在暗面空間中將游標 hover 於帶有 `soul === 1` 之建築物上方（觸發 `ondarkhover`），即可吸取其靈魂轉化為對應 `soulPower` 之真實能量（`Reality`），用於升級 Strange3（需 2,048 Reality）與開闢反轉深淵（The Inverse Chasm，需 4,000 Reality）。

## 13. Strange2 與虛空雕塑雙落成、路標暗面傳送與反轉深淵（The Inverse Chasm）終局架構

- **里程碑達成**：
  - **Strange2（Hollow Rock Facility）**：成功於 `(-100, -100)` 升級完成！
  - **Void Sculpture（虛空雕塑）**：成功落成於 `(8, 2)`（造價 1,024 Void）。
- **暗面空間（Plane 1）首度啟動與路標機制**：
  - 虛空雕塑落成後，需建造路標（**Waypoint**，造價 52.4 萬 Charonite、26.2 萬 Qanetite、26.2 萬 Beta-Pylene、8 空心石）。
  - 對路標進行互動（消耗 512 Hell Gem），觸發通往暗面（Dark Plane，`plane = 1`）的首次維度躍遷。
  - 暗面空間中所有建物帶有 `soul === 1`，透過吸收靈魂產出 **Reality（`resources[9]`）**。累積達 512 Reality 後，點擊暗面雕塑返回光面，並將雕塑永久啟動為雙向傳送門（`bridge = true`）。
- **反轉深淵（The Inverse Chasm，`chasm`）機制全解析**：
  - **定義與背景**：「通往未知之橋（A bridge to the unknown）」，直通終局維度裂隙的超巨型終極引力結構。
  - **解鎖條件**：`entitiesInGame.strange3 > 0`（Strange3 造價：2,048 Hell Gem、128 Hollow Stone、2,048 Void、2,048 Reality）。
  - **造價**：2,000 萬 Charonite、2,000 萬 Elmerine、2,000 萬 Qanetite、2,000 萬 Beta-Pylene、4,000 Hell Gem、20,000 Chromalit、200 萬 Celestial Foam、4,000 Hollow Stone、4,000 Void、4,000 Reality。
  - **深淵傳導網路（Chasm Network）**：
    - 反轉深淵落成後，會透過 **Conductor（導體 / 雙向信號槽）** 與全場的 **Industrial Silo MKII（`silo2`）**、**General Decay Reactor（`generaldecay`）** 以及後續的 **Gradient Well（漸變井）** 連接成統一傳輸鏈網。
    - 漸變井產出的資源將不再需要人工或普通物流，直接透過深淵通道（`createChasmTransfer`）向反轉深淵匯聚注入。
- **終極通關鏈條（Singularity Collapse to Victory）**：
  - $\text{Strange2} \to \text{Void Sculpture} \to \text{Waypoint / Reality} \to \text{Strange3} \to \text{The Inverse Chasm} \to \text{Gradient Well} \to \text{Pinhole (針孔)}$。
  - 針孔落成後引發奇點坍縮事件（Singularity Event，播放結尾音樂 `shallow_anne`，釋放宇宙閃電摧毀全工廠直至 `stuff.length < 3`），點擊北方奇異岩石觸發 `watchCredits()` 正式通關！





### 14. Phase 14: Hollow Rock Cultivation (Flowers, Fruits) & Inverse Chasm Progress (2026-10-31)
- **Flower & Fruit Automation**:
  - Hollow rocks around (-100, -100) are automatically upgraded to Flowers (flower, soulPower: 32) and Fruits (fruit, soulPower: 128).
  - Strange3 periodically feeds seeds to Fruits (fruit.seed()), which grow and directly yield Hollow stones via resource transfer into storage.
  - Plane 1 sweeping across Flowers & Fruits rapidly scales Reality harvest.
- **Foam & Chromalit Governance**:
  - converter76 consumes 8,192 Celestial foam per cycle and outputs 2,048 Chromalit + 1 Hell Gem.
  - Keeper v7.9 limits converter76 refill once Chromalit >= 20,000, allowing all Celestial foam cubes to accumulate toward the 2,000,000 requirement for The Inverse Chasm.
  - Hollow stone target raised to 4,200 (breaking hollows and harvesting fruits).
- **Chasm Readiness**:
  - 5/10 materials already met (Charonite, Elmerine, Qanetite, Beta-Pylene, Void).
  - Active accumulation in progress for Celestial foam, Hollow stones, Hell gems, and Reality.

- **Flower Price Scaling Discovery & Cap**:
  - Discovered Hollow flowers have exponential price scaling (40th flower cost ~1,469 Hollow stone & 11,758 Void).
  - Capped flower planting at current 40 units in Keeper v7.10 to protect Void and Hollow stone reserves for The Inverse Chasm (both require 4,000).
  - Void and Hollow stone production actively regenerating via Annihilator and Fruit cultivation.
