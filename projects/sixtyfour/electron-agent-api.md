---
title: Sixty Four 的 Electron agent API 接入與原生操作語意
scope: project
project: sixtyfour
status: active
updated: 2026-10-30
---

# 接入方式

Windows 遊戲版本 1.2.1 的 Electron 程式碼未封裝於 ASAR。`<game-install>/win-unpacked/resources/app/main.js` 載入 `game/index.html`，renderer 使用 `contextIsolation: false`、`nodeIntegration: true`。頁面頂層宣告 lexical `let game`，可以透過 `webContents.executeJavaScript` 存取；它不是 `window.game`。

已實作並實機驗證的方式是在 main process 載入 bridge，提供只綁定 loopback、有認證且限定命令的 HTTP API，再由 main process 呼叫 renderer adapter。保留命令白名單，禁止任意 JavaScript evaluation 輸入；不必向 agent 暴露 CDP。CLI 與 HTTP 共用相同 adapter，並序列化所有遊戲操作。安裝前備份原檔和 profile，Steam 更新後需要檢查 hook 是否仍存在。

# 必須保留的遊戲規則

- `game.stuff`、`stuffMap`、`resources`、`codex.entities` 提供世界與目錄資料；`entityAtCoordinates([x,y])` 查格子。
- 放置與升級走 `pickupItem(type)`、設定 `hoveredCell` / `hoveredEntity`，再 `processClick()`。不要直接呼叫 `addEntity`，否則會跳過成本與副作用。
- `processClick()` 本身不檢查商店解鎖和唯一性。Adapter 必須另驗證 `canPurchase`、`unlockedEntities[type]` / `shouldUnlock(game)`、`onlyones`、目前 plane、`canAfford`、佔位與升級目標，並確認放置後的實體。
- 互動沿用 entity 的 `onmousedown` / `onmouseup` 或 dark callback；先檢查 `canHit()` / `canDarkHit()`。重複點擊每次重新檢查，cube 破裂後停止，並維持正常 `stats.totalCubeClicks`。Pump 長按期間需要 `game.hoveredEntity === target`，否則其 `update()` 立即釋放；結束時必須 `onmouseup()`。
- 長按不能只每 25 ms 重設 hover：`updateTranslation` / `processMousemove2` 的原生游標刷新會在兩次重設之間改寫 `hoveredEntity`，讓 Pump 提前取消。已驗證的修正是在長按期間暫時包裝目標的 `update`，每次呼叫原生方法前先設定 `game.hoveredEntity = target`；在 `finally` 恢復原本 own / prototype 方法關係並釋放按鍵。VM 測試重現游標覆寫後通過，實機長按 5 秒的 pump 深度由 0 增至約 0.795 並生成 cube；後續正常點擊破壞 cube、回收及區域清理均驗證完成。
- 回收依 `eraserType` 選 `eraser` / `eraser2` / `eraser3`，先跑 `canEraseNow()` 再走 `processClick()`；不可直接 `clearCell()`。這樣才能保留 cube / indestructible / 最後一台 excavator 的保護、回收費用、退款和統計。
- 退款沿用原生 `getRealPrice(name, true)`。函式在倍率 `mult === 1` 時直接回傳基礎價格，未套回收折扣；勿自行重算成固定百分比。退款通常經 VFX 非同步入帳，回收回應後應稍後再讀資源。
- 操作後還原暫時使用的選取、手持物與 hover 狀態，避免把已刪除 entity 的參照還原。Menu 開啟時明確拒絕世界操作，另提供 `resume` 關閉 splash。

# 存檔與驗證邊界

`saveLoop()` 每 10 秒呼叫 `saveGame()`。存檔是 base64 編碼的 JSON，localStorage key 為 `abstractv03` 加目前 Steam 帳號識別；Electron profile 為 `%APPDATA%/sixtyfour`。`saveGame()` 同時透過 IPC 要求 Steam Cloud 寫入 `SFSaveFile`，但沒有完成 acknowledgement；API 只能確認 localStorage 已寫入、cloud request 已送出。`preventSaving` / `preventCloud` 生效時必須如實拒絕保存。

正常遊戲選單退出先 `game.saveGame()` 再 `game.spaceport.send('quit', '')`；main process 的退出 handler 本身沒有額外保存。要重啟安裝 hook，先用正常退出流程並備份存檔，避免直接強制終止。

已驗證 state、catalog、resume、place、interact、remove、save 的完整實機流程，以及 adapter 的解鎖、成本、佔位、最後 excavator 保護、hold 釋放與存檔限制測試。實機回收也確認原生倍率為 1 時退回基礎價格。Steam Cloud 遠端持久化完成仍無法由現有遊戲 IPC 證實。

# OpenSpeedy 降速後 freeze 的計時修正

只加速 renderer 時，OpenSpeedy 從 100 倍降至 10 倍的實機測試曾觀察到 `performance.now()` 倒退，`game.time.lt` 留在未來，而 `renderTime.lt` 仍隨畫面前進。這時模擬 worker 的 tick 停住，統計不再增加，但 renderer 與 rAF 還活著。不能只在 `updateLoop()` 裡加 delta 下限：停住的 worker 根本不會再呼叫它。

已驗證的最小修正是在 `renderloop()` 檢查有限的當前時間是否早於 simulation 或 render 的上一個時間戳；發現倒退時先重設兩個基準，`terminate()` 舊 clock，再照原生方式建立 `new Worker('scripts/clock.js')`，為 `message` 綁回 `this.updateLoop()`。正常畫面不重建 worker。simulation delta 同時檢查有限值並限制在 0～1000 ms；render delta 檢查有限值與非負值，但保留正常正值，不任意加上上限而改變加速行為。

另一個已用原生程式重現的失敗路徑是負 simulation delta：`Pump.update()` 可將 `spoolup` 減到負值，`Sprite.renderState()` 再以負 frame index 取到 undefined mask，反覆拋錯。simulation 與 render 使用不同時間戳，兩邊都必須防守。原生更新例外仍須保留可見，不能靠吞掉所有錯誤假裝恢復。

驗證要區分層次：載入原始 `Game` 程式的 VM 測試，已確定受控倒退會只重建一次 worker、下一個 tick 恢復正值更新並送出 acknowledgement；正常／加速正 delta、非有限值與原生例外路徑均有覆蓋。安裝修正後的真實 100→10 切換未再 freeze，後續 10 倍的連續兩次讀取也確認 simulation 與遊玩時間持續前進；但該次實機切換時時鐘保持單調、recovery counter 為 0，因此不能宣稱實機觀察到 worker 重建，該分支的證據來自受控原生程式測試。

修正針對已確認的遊戲 1.2.1 `game.js` SHA-256；版本或 hash 不符時拒絕套用。clock fix 與 API bridge 各自記錄原檔、修改後 hash 和還原路徑。安裝時須處理「遊戲檔已改、安裝紀錄寫入失敗」的狀態：已用唯讀紀錄檔與套用後才鎖住紀錄檔的 fixture 重現，修正後會保留舊紀錄、恢復經 hash 驗證的原檔並留下備份。還原也接受已恢復成原始 hash 的檔案，讓中斷後重試能繼續完成。

# 正常 Steam Play 會載入磁碟上的修正

已核對 AppID `2659900` 的安裝資訊與 Steam 快取啟動設定：Windows 執行檔為 `win-unpacked/sixtyfour.exe`，沒有 per-app LaunchOptions 覆寫。遊戲載入鏈是 `main.js` → `game/index.html` → `scripts/game.js`，所以修改的就是正常 Steam Play 使用的原檔，不需要另外的 launcher。實機另經保存、正常關閉、按 Steam 綠色 Play 重新啟動，確認新遊戲由 Steam 建立、API 自動啟動，並且 simulation 連續前進。

`state.data.timing.clockFixActive` 是唯讀診斷，檢查目前遊戲實際使用的 `game.renderloop` 函式是否含 clock-fix marker；正常 Steam Play 重啟後已讀到 true。它補足磁碟 hash 檢查，能確認執行中的函式已載入補丁。這不代表 Steam 更新或驗證檔案後仍保留修改；兩者可能覆蓋原檔，屆時需重新檢查與安裝。

# 背景／遮蔽視窗下的批次自動化注意事項

- **避免在視窗背景／遮蔽時傳入 `count >= 5` 或 `holdMs > 0`**：`renderer-adapter.cjs` 的 `interact` 在 `count > 1` 或 `holdMs > 0` 時會於 renderer 主執行緒內循環呼叫 `window.setTimeout`。當遊戲視窗處於背景、最小化或被其他全螢幕視窗遮蔽時，Chromium 的 **Intensive Wake Up Throttling** 會在 `setTimeout` 巢狀層數 `nesting_level >= 5`（即未提前擊碎方塊且 `count >= 6`）時，將後續每次 `setTimeout` 節流為 **每分鐘只喚醒一次（對齊每分鐘固定秒數）**，導致 `bridge-main.cjs` 的單執行緒序列化佇列（`pending: 1`）卡住數分鐘。
- **建議呼叫方式**：批次腳本控制時，`interact` 一律使用預設單擊（`count: 1, holdMs: 0`），此時 renderer adapter 完全同步執行、不觸發任何 `window.setTimeout`；若需連點或等待 VFX 入帳，將迴圈與 `sleep` 放在外部 Node.js 呼叫端處理。此外，長時間連續發送 HTTP 請求的腳本建議加上 `Connection: close` header，避免 Node `fetch` 重用 keep-alive 連線時撞上 `server.requestTimeout = 10000` 而發生 `ECONNRESET`。
