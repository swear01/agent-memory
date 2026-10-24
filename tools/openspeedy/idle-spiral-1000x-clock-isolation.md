---
title: Idle Spiral 100–1000×：原生時鐘隔離、宏步與跟著倍率的真實時間系統
scope: tools/openspeedy
tool: openspeedy
tags: [idle-spiral, harmony, unity-mono, steam, performance]
status: active
updated: 2026-10-24
---

Idle Spiral 3.0.82 build 117（Unity／Mono）＋ OpenSpeedy 3.3.8 `speedpatch64.dll`。延續 [idle-spiral-simulation-budget](idle-spiral-simulation-budget.md)；這份記錄把目標從 20× 推到 100–1000× 時確認的根因、做法與量測。所有補丁都是 Harmony 2.4.3 執行中補丁，經 Steam 自動載入鏈安裝，原遊戲組件不改。

## 卡死根因：原生等待 hook 影響 steamclient

speedpatch64 以 MinHook 掛 Sleep／WaitFor*／QPC／GetTickCount 等。等待時間會除以倍率並截斷，所以 100× 時任何小於 100 ms 的等待都變成 0。舊局的 7–14 秒停頓位於 `SteamAPI_ManualDispatch_GetNextCallback`。

ClockIsolation 把原生 `SP_IsEnabled` 讀的資料槽改指向一個恆為 false 的位元組，讓原生時鐘與等待回到 1×；託管模擬自己讀原本的旗標加上 `SP_GetSpeed`，只套一次倍率（上限 1000）。**從啟動就隔離**後，多次 300 秒 100× 觀測與 400× 連續 15 分鐘（61 個取樣全有回應）都沒有再出現停頓，Steam 回呼最長約 6 ms。這支持「原生 hook 影響 steamclient」，但 Steam 內部確切機制未證實。

## OpenSpeedy 開關語意

- `SP_Install`（DLL 載入時）會把**宿主程序設為啟用**，所以自動載入的遊戲即使沒在 GUI 開關也會用共享倍率。自動載入時若 DLL 是我們自己載入的（之前沒有 `GetModuleHandle("speedpatch64.dll")`），就立刻呼叫 `SP_Disable(pid)`；若 OpenSpeedy 早已注入就保留原狀態。
- 倍率存在 DLL 的共享段（跨程序）；每程序開關在 file mapping `OpenSpeedy.<pid>`。GUI 只有在選取該程序時才讀真實狀態；直接關掉 OpenSpeedy 不會停用遊戲。
- 用 `SP_SetSpeed` 直接寫共享段測試後，GUI 顯示值不會跟著變；使用者會看到「設定 100 實際 400」。測試後一定要還原。

## 吞吐量：每秒約 5000 次 Advance

`IdleSpiralSystem.Advance` 每次約 50–90 µs（視遊戲進度與面板），單執行緒每秒約 5000 次就飽和。MacroStepGuard 在倍率 >100 時每次 Advance 推進 k=⌈倍率/100⌉ 個 0.02 秒步（gameDelta／baseDelta 都乘 k），每次上限 0.4 遊戲秒（等於遊戲 Offline Boost 的步長，Boost 開啟時退回 k=1）。原 runtime 的積壓上限 40 遊戲秒必須一起放大成 40×k；否則 1000× 在背景 20 FPS 時每幀要 50 遊戲秒，只跑到 799×。

遊戲設定 `currencyUpdateTick` 從 4 改 20（呼叫遊戲自己的 `ApplyCurrencyUpdateTick`），貨幣結算 15→4 µs／步。

## 隔離後「慢下來」的真實時間系統

原生時鐘回到 1× 後，原本靠被加速時鐘的系統會變慢，要逐一處理：

- **自動購買／自動轉生**：四個 automation controller 每 1 秒 `Time.realtimeSinceStartup` 才 Tick。改餵依倍率前進的時鐘（每幀最多一次，和以前時鐘被加速時一樣）。自動轉生觸發的 `PrestigeCheckpoint.TrySave` 會完整寫檔，限制每 5 真實秒一次。
- **背景 20 FPS**：`PcFrameRateController` 失焦時限 20 FPS，1000× 下每幀 50 遊戲秒、畫面很卡。patch `ApplyResolvedCadence` 的 prefix 把 focused 設為 true；不要 patch `ResolveTargetFrameRate`，它太小，可能被 Mono JIT inline。量 FPS 時要用 `GetForegroundWindow` 確認焦點，否則會把 20 FPS 上限誤當效能問題。
- **公式 HUD／Omega 顯示**：每 0.1 真實秒才刷新，是正常行為，不是 bug。
- **Daily Spiral**：在真實本地午夜由 `DailySpiral.EnsureCurrentDay` 重置；CalendarGuard 刻意讓日期維持真實時間。若存檔 `dailyLastResetDate` 在未來（舊的時鐘超前留下的），會一直鎖到真實日期追上。每累積 86400 個「多出來的遊戲秒」（`(倍率-1)×真實秒`），就把 `dailyLastResetDate` 清成 0，再呼叫 `EnsureCurrentDay`，讓遊戲寫回真實今天的日期：存檔永遠不會有未來日期。Season 跑局排除。面板倒數 `DailyBonusScreenController.SecondsUntilLocalMidnight` 回傳真實秒，和遊戲秒比較前要先**乘以倍率**；第一版直接比較，結果一重置就又顯示約 10 小時的真實午夜。

- **Theory 自動研究**：每 1 真實秒升 100×TickMultiplier 級。`FirstTheoryTransaction.LevelUpByLevel` 逐級迴圈，每級約 7 µs，所以單純調高上限會卡幀。迴圈中等級最後才寫回，每級都付起始價格，可改成一次扣 `單價×min(級數, floor(資源/單價))`，扣款與原迴圈一致，每次約 45 µs。使用者要「幾乎無上限」：高於 1× 時每批只受資源限制（安全上限 1e15），一批就 +6600 萬級，之後由 Inspiration 收入決定速度。
- **其他 Theory 的 Auto Study**：原遊戲只有 Pythagorean 有開關。同一個開關元件（`TheoryUIManager.AutoTheoryUpgradeToggle`／`ToggleParent`）在所有 Theory 都會顯示，狀態存在 `theory-auto.txt`，不寫進存檔，預設開啟；Pythagorean 的協程要改讀存檔旗標，否則切到其他 Theory、關掉開關時，它也會停。EXP 滿了就呼叫 `TryLevelUp`。每個模擬步檢查一次，實測每真實秒約 8 ms；改成每幀一次只要約 0.1 ms。面板沒開時，可以從 `GameSystem.Update` postfix 用 reflection 呼叫 `RefreshPresentation`／`StoreAutoTheoryUpgradeToggle` 來驗證 UI，不必操作畫面。

## 戰鬥自動開始：目標低於 Nuke 起始難度

`reward.difficulty = net_difficulty + Nuke 週期`。自動開始目標難度（`inputDifficulties`，多半是 1）低於起始難度時，每贏一批就重開，1000× 約每秒 1130 次。玩家看到的是難度數字一直跳。每次重開在同一步內 battle→quitting→battle，`BattleCtrl.StateChanged` 的兩個 UI 訂閱者（`BattleUIManager.OnBattleStateChanged`、`Battle_Mono.ApplyBattleState`）每次都切換 Doozy 結果／戰鬥 view 與按鈕，`StartBattle` 每次約 189 µs。把這兩個 handler 延到 `GameSystem.Update` postfix、用最後狀態每幀呼叫一次後，451×→998×，`UpdateBattle` 每步 283→98 µs。剩下的成本是重開本身（每批新建敵人 `EnemyContainer.GetEnemy` 約 19 µs），這種模式下 1000× 約 21 FPS，屬遊戲規則成本。

開關本身會正常存檔（`setSaveKey` → explicit-save）。使用者說「開關會跳」時，先用探針確認，再問清楚現象，才不會修錯地方。

## 量測與診斷方法

- 用 `QueryUnbiasedInterruptTimePrecise`（KernelBase.dll）量真實時間；被 hook 的 QPC／Stopwatch 不可信。
- 每個臨時探針用**不同的 assembly name**；Mono 會重用同名 assembly 的靜態狀態，重複 apply 會累加或失效。正式 guard 同名熱換也不會生效，要關遊戲後用安裝器 Upgrade。
- 探針參考 `BigDouble.dll` 曾在加了 HintPath 後仍報 CS0012；改用 HarmonyLib `Traverse`／`object` 讀值即可，不必強型別。
- SimulationProbe 每 127 次取樣一次 Advance，可列出各階段成本；搭配 adaptive／macrostep 狀態檔的計數差，才知道實際每秒呼叫次數。兩者不一致時，以計數差加 frame probe 為準。
- 狀態檔每 128～1024 幀才寫一次，卡住時可能是舊資料。

## 時鐘偏移與最終封存

2026-10-24 量測（啟動後約 30 分鐘）：`GetSystemTime`、`GetSystemTimePreciseAsFileTime` 與 CalendarGuard 的 `UtcNow` 都是真實時間；只有被 OpenSpeedy hook 的 `GetSystemTimeAsFileTime`（也就是 Mono 的 `DateTime.UtcNow/Now`）固定快 17 分 41 秒（1061 秒），而且不再增加。遊戲自己的 53 個日曆呼叫點經 CalendarGuard 走真實時間，所以這個偏差只影響未替換的 `DateTime.Now` 使用者（Mono、Unity、其他 DLL、探針）。1061 秒 ÷ 999 ≈ 1.06 秒，推測是隔離生效前，hook 以 1000× 跑了約一秒，之後一直保留連續偏移。哪一段造成這一秒（`LoadLibrary`→`SP_Disable`，或 GUI 先注入）未查清；使用者刪除遊戲後，此項停止。

使用者已刪除遊戲。最終程式碼（9 個 branch，預設 `hundred-performance`）公開封存在 <https://github.com/swear01/idle-spiral-openspeedy-patch>，Release `archive-2026-10-24` 附安裝包 zip，SHA-256 `c3de77843bb3592c4cef7a663b934a6e5a692f463974e580b92531d183fe5f39`（README 重寫後重新上傳；tag 仍指向程式碼 commit 1ec84a1，README 改寫是 5803a43），已以匿名下載核對雜湊。使用者要求 README 白話易懂：先寫是什麼、修了什麼、安裝／使用步驟，再放原理；「沒有補丁時」的對照要依實測，不要寫推測的數字。安裝包刻意不含 `verification/` 本機日誌（內有本機路徑與自測 token 字串）、遊戲檔案與 speedpatch64.dll。
