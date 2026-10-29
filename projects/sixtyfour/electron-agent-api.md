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
