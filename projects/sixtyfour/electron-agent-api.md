---
title: Sixty Four 的 Electron agent API 接入與原生操作語意
scope: project
project: sixtyfour
status: active
updated: 2026-10-31
---

# 接入方式

Windows 遊戲版本 1.2.1 的 Electron 程式碼未封裝於 ASAR。`<game-install>/win-unpacked/resources/app/main.js` 載入 `game/index.html`，renderer 使用 `contextIsolation: false`、`nodeIntegration: true`。頁面頂層宣告 lexical `let game`，可以透過 `webContents.executeJavaScript` 存取；它不是 `window.game`。

已實作並實機驗證的方式是在 main process 載入 bridge，呼叫 renderer adapter；HTTP API 最初也在 main process，後續已移至未加速的獨立 Node 程序（見下文）。HTTP 只綁定 loopback、有認證且限定命令，禁止任意 JavaScript evaluation 輸入；不必向 agent 暴露 CDP。CLI 與 HTTP 共用相同 adapter，並序列化所有遊戲操作。安裝前備份原檔和 profile，Steam 更新後需要檢查 hook 是否仍存在。

# 必須保留的遊戲規則

- `game.stuff`、`stuffMap`、`resources`、`codex.entities` 提供世界與目錄資料；`entityAtCoordinates([x,y])` 查格子。
- 放置與升級走 `pickupItem(type)`、設定 `hoveredCell` / `hoveredEntity`，再 `processClick()`。不要直接呼叫 `addEntity`，否則會跳過成本與副作用。
- `processClick()` 本身不檢查商店解鎖和唯一性。Adapter 必須另驗證 `canPurchase`、`unlockedEntities[type]` / `shouldUnlock(game)`、`onlyones`、目前 plane、`canAfford`、佔位與升級目標，並確認放置後的實體。
- catalog 保留原生名稱、描述與 `getRealPrice(type)` 的即時 indexed price，另以 state 相同的 resource index／名稱加入非零 `materials: [{index,name,amount}]`；缺名稱時用 `resource-N`。`placementSupported` 必須與 place 的限制一致：eraser 系列及沒有 class 的項目為 false，且 `available` 為 false；商店可購買不代表可透過 place 放置。可放置項目的 `available` 仍只是目前 plane、解鎖、資源和唯一性資格，實際座標仍由 place 驗證。
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

驗證要區分層次：載入原始 `Game` 程式的 VM 測試，已確定受控倒退會只重建一次 worker、下一個 tick 恢復正值更新並送出 acknowledgement；正常／加速正 delta、非有限值與原生例外路徑均有覆蓋。安裝修正後初次真實 100→10 切換未再 freeze，後續 10 倍的 simulation 與遊玩時間持續前進；該次時鐘保持單調、recovery counter 為 0，不能說該次觀察到 worker 重建。後續實機倍速矩陣讀到 `rollbackRecoveries: 1`、`clockFixActive: true`，模擬繼續更新，才補足實際時鐘倒退恢復的證據。

修正針對已確認的遊戲 1.2.1 `game.js` SHA-256；版本或 hash 不符時拒絕套用。clock fix 與 API bridge 各自記錄原檔、修改後 hash 和還原路徑。安裝時須處理「遊戲檔已改、安裝紀錄寫入失敗」的狀態：已用唯讀紀錄檔與套用後才鎖住紀錄檔的 fixture 重現，修正後會保留舊紀錄、恢復經 hash 驗證的原檔並留下備份。還原也接受已恢復成原始 hash 的檔案，讓中斷後重試能繼續完成。

# 正常 Steam Play 會載入磁碟上的修正

已核對 AppID `2659900` 的安裝資訊與 Steam 快取啟動設定：Windows 執行檔為 `win-unpacked/sixtyfour.exe`，沒有 per-app LaunchOptions 覆寫。遊戲載入鏈是 `main.js` → `game/index.html` → `scripts/game.js`，所以修改的就是正常 Steam Play 使用的原檔，不需要另外的 launcher。實機另經保存、正常關閉、按 Steam 綠色 Play 重新啟動，確認新遊戲由 Steam 建立、API 自動啟動，並且 simulation 連續前進。

`state.data.timing.clockFixActive` 是唯讀診斷，檢查目前遊戲實際使用的 `game.renderloop` 函式是否含 clock-fix marker；正常 Steam Play 重啟後已讀到 true。它補足磁碟 hash 檢查，能確認執行中的函式已載入補丁。這不代表 Steam 更新或驗證檔案後仍保留修改；兩者可能覆蓋原檔，屆時需重新檢查與安裝。

# API 的真實時間隔離與背景批次操作

舊版 `interact` 在 renderer 內用 `performance.now()`／`setTimeout` 控制長按與連點，因此受到 OpenSpeedy 倍率、時鐘倒退及 Chromium 背景排程影響。實機曾在 10 倍、視窗無 MainWindowHandle 時，`count:16` 讓操作佇列卡住約 15 分鐘後自行恢復，期間存檔與模擬仍前進；背景 timer throttling 是可能原因，未經 Chromium trace 證實。舊版可用外部呼叫端逐次單擊避開 renderer timer。當時亦曾建議 `Connection: close` 排查連線重用，但沒有單獨證實 `ECONNRESET` 根因。

API v2 把 HTTP、認證、容量 16 的操作佇列、8 秒 deadline、連點的 30 ms 間隔與長按等待移到 **standalone Node.js 24+**。Electron main 透過 `child_process.fork`、明確 `execPath: config.nodePath`、空 `execArgv`、隱藏視窗與 IPC 啟動 `bridge-gateway.cjs`，自己只驗證命令並執行 renderer adapter；不是 `utilityProcess`。獨立 Node 以 `process.hrtime.bigint()` 計算 elapsed，API 回應中的遊戲 `performance.now()` 僅供診斷。必須讓該 Node 程序不被 OpenSpeedy 注入；中間版曾核對遊戲 main／renderer／utility 載入 speedpatch，而 gateway 未載入，後續另隔離遊戲本身的原生時鐘（見下文）。只換成另一個仍被注入的程序，不構成時鐘隔離。

長按透過私有 `beginInteraction`／`endInteraction` 跨 IPC 配對，外部 gateway 計等待時間，renderer 只維持與釋放目標；私有命令不對 HTTP 開放。deadline 回覆 504 不等於原生操作已取消：已送進 renderer 的工作仍可能稍後完成，佇列須等該工作及清理 acknowledgement 才能執行下一個，過期的排隊工作則不能再開始。遊戲忙碌時 health 仍可直接回覆。退出、gateway 斷線或 child 結束時要 best-effort 釋放互動並關閉外部服務，不使用無限重啟迴圈。

API 隔離階段的 1000 倍壓力測試又找到缺口：main executor 曾殘留 15000 ms 的 `setTimeout`／`Promise.race` 與 `rendererTimeoutMs`。實機 `beginInteraction` 在約 2.285 真實秒回覆 `RENDERER_TIMEOUT`；該 timer 也處於被加速程序，原生 RPC 卻仍可能完成，gateway 若視它已結束就會過早推進操作佇列。修正完全移除 main 的 timer／race／timeout 設定，直接回傳實際 `executeJavaScript` promise；唯一 deadline 留在外部 gateway，回覆 504 後仍須等原生 RPC 與清理真的結束。回歸測試把遊戲程序所有時鐘／timer 設為一呼叫就拋錯，並用 deferred RPC、重複訊息與晚發生的原生錯誤驗證生命週期；當時 main 10／10、全套 56／56 通過且無 skip。該修正版經 Steam 重開，但仍不是遊戲原生時鐘隔離的驗證。

隔離保證的是等待時鐘與 HTTP 排程不隨遊戲倍速縮放，**不保證硬即時的按鍵釋放或回應**。實機 `holdMs:1000` 在 1／10 倍的總回應約 1055 ms，1000 倍時約 4014 ms，差異包含 renderer IPC 排隊。回應的 hold timing 要分開列 requested hold、外部 hold wait、begin／end round trip；受控測試以 begin 250 ms＋wait 1000 ms＋end 600 ms，證明總耗時 1850 ms，不能把 requested hold 當成總回應或實際世界持續時間。

安裝器記錄絕對 `nodePath` 並驗證 standalone Node 24+；正常 Steam Play 自動啟動 gateway。升級既有 API 時先保存並正常關遊戲，備份整個舊 bridge、CLI config 與 installation record，只更新 bridge 與兩份 config 的 runtime 路徑，保留 port、認證及原生 hook 的 hash／還原資訊。已用隔離 fixture 驗證全新安裝、舊版升級、metadata 唯讀拒絕、晚發生的 config 寫入失敗回滾，以及可重試還原。Node 更新若移除原絕對路徑，需重新檢查設定；Steam 更新／驗證檔案亦可能覆蓋遊戲 hook。

# 唯讀延遲監測：fetch 路徑不等於伺服器處理時間

Windows／Node.js 24.19.0 的同一個 client 程序，以同一本機 `/health`、未修改的 gateway 做循序唯讀比較，各 30 次皆零錯誤：fetch burst 的 median／P95 約 15.475／16.523 ms；原生 `node:http` keep-alive 約 0.267／0.598 ms；原生 HTTP 每次新連線約 0.807／1.017 ms。另各 30 次、間隔 200 ms 的取樣，fetch median 約 16.278 ms，原生 HTTP keep-alive 約 0.714 ms；伺服器自己回報的處理時間 median 約 0.01 ms。同程序再交錯呼叫 fetch／原生 HTTP 各 30 次，median 約 15.129／0.379 ms、皆零錯誤，支持當時額外延遲與 fetch client 路徑相關，並非該 API 無法避免的本機網路或 handler 成本。該輪尚未定位 Undici 內部等待，不能外推為所有 Node 版本／環境的通則；後續 source／trace 定位見下節。

fetch 並非永遠需要約 16 ms：另一程序曾在明確指定 Connection header 時量到約 1 ms，但後續同程序交錯測 default fetch／明確 keep-alive／帶 AbortSignal，各 30 次 median 仍約 15.64／15.51／15.47 ms，沒有可靠重現 header 可解決問題。故只能記錄目前環境及呼叫條件下反覆出現的差異；`Connection: close` 或 `keep-alive` 都不是已證實的修復。

同次原生 HTTP keep-alive 的 state 取樣排除前兩次 warm-up 後，18 次 median 約 3.951 ms、最慢 5.764 ms；較早的 fetch state 49 次 median 約 20.309 ms、最慢 24.209 ms。state 是不同窗口，不能當成相同遊戲狀態的確定性重播；較乾淨的 client A/B 證據來自上述相同 health endpoint。較早逐次啟動 CLI 的完整耗時約 100～127 ms，包含程序初始化、設定、HTTP 與輸出，不能當成純程序啟動成本，須與單獨 HTTP latency 分開。該輪僅監測，當時沒有修正或更換 client，也沒有修改遊戲、server、CLI 或設定；短期循序取樣不等於並行負載測試。

# 低延遲 client 與 Antigravity 的常駐終端入口

後續在 Node.js 24.19.0 隨附的 Undici 7.29.0 找到 `scheduleIdleSocketValidation` 的 `setTimeout(callback, 0)` 路徑；`resumeH1` 等它完成才重用 idle socket。隔離 fixture 交錯 fetch／原生 HTTP 各 40 次，fetch 每次都有 idle validation，timer median 約 13.675 ms、request create 到 send headers 約 13.840 ms，server work 僅約 0.017 ms；全程重用同一連線。這把該輪主要延遲定位到 headers 送出前的 client 排程，沒有證明 OS kernel 根因。版本 source 對照與可重用驗證方式記在 [Node HTTP 延遲筆記](../../tools/node/http-client-latency.md)。

專案新增無 npm 相依的 `client.mjs`，用 `node:http`／`node:https` keep-alive Agent。SDK 在 agent 主循環外建立一次，結束時 `close()`；保留固定路由與認證、涵蓋 socket queue／headers／body 的完整 wall-clock deadline、8 MiB 回應上限及關閉清理，不自動重送 mutation，不覆寫全域 timer 或 patch Undici。

Antigravity 等已有終端工具的 agent 使用 `node <package-root>/sixtyfour.mjs session` 一次啟動並保持 stdin；後續透過同一個終端程序寫入 `{"id":1,"argv":["state"]}` 等 NDJSON。每行回覆一個附相同 id 的 JSON，錯誤不終止後續行；按序執行、單行上限 16 KiB，EOF 完成已輸入工作後清理，SIGINT／SIGTERM 中止 client 並放棄尚未輸出的結果，避免 stdout backpressure 卡住終止。這個入口不另監聽埠、不需要 MCP，也不需要 agent 規則或全域 app 設定。保留一次性 CLI，但每次另啟 Node 仍會重做程序初始化與首筆請求；持續使用要把 Node process 和 HTTP pool 一起重用。

升級後的新遊戲確由 Steam 父程序建立並自動載入新 renderer；該輪是外部 Steam 活動重新啟動，並非手動點 Play 的新證據。實際 catalog 58 項皆有有效原生說明與 named materials：55 項支援放置，3 個 eraser 工具的 `placementSupported`／`available` 為 false。當時 OpenSpeedy 保存倍率為 1000、新程序原旗標 Off，有效倍率仍為 1；本次量測沒有改變保存倍率，不可拿來當已啟用 1000 倍的效能測試。

最終輸出處理修正後的唯讀 benchmark，SDK health 30 次 median／P95／max 約 0.470／0.604／0.732 ms；SDK state 30 次約 4.803／11.710／179.450 ms，server median 3.694 ms，逐筆扣除 server elapsed 的 client residual median 約 0.839 ms。session 的 health 15 次、state 7 次、catalog 8 次 median 分別約 0.932／4.956／9.601 ms，全部零錯誤；catalog 最慢約 212.507 ms，包含 server outlier，materials 也通過。session 第一筆完整回覆約 114.927 ms，包含初始化、設定、首筆 HTTP 與輸出，不能叫純啟動時間。消除 client 固定等待後仍有遊戲／server outlier，不能承諾每筆 state 都小於 5 ms；以上亦非長期或並行壓測。

最終全套 Node 測試 95／95 通過、無 skip，包含 client 13、CLI 9、renderer 20 及其餘 53 項。CLI 的 2 MiB stdout backpressure 測試覆蓋 SIGINT／SIGTERM handler：Windows 使用非同步 `fs.write()` 使 handler 可執行，先關閉 client，再只以 SIGKILL 終止自身 PID、放棄未輸出結果；EOF 則正常排空。Windows 該路徑 exit code 為 1，Unix 為 130／143。不能把 `process.stdout.write()` 的 backpressure 當成 API 卡住，也不能假設 `process.exit()` 一定不等待被堵住的 pipe worker；可重用限制見上述 Node 筆記。

# 遊戲原生時鐘隔離：只在 OpenSpeedy 明確啟用時縮放模擬

API 隔離後使用者再次回報畫面凍結；當時 state 中 simulation 仍前進，但 `renderLast` 固定、rAF 不再更新。這與先前「rAF 活著、worker 停住」相反，僅放在 renderloop 裡的 rollback recovery 無法救回沒有執行的 renderloop。**HTTP／simulation 活著不能當畫面正常的證據**；後續驗證另檢查 render 時鐘，並用實際 canvas 連續畫面與資源顯示確認畫面改變。沒有證明 Chromium／驅動內部的確切停頓原因。

目前改採 Idle Spiral 已驗證過的架構模式：讓 OpenSpeedy 的原生 clock／wait hook 實效維持 1 倍，只讀原來的控制旗標與共享倍率，於遊戲自己的 dt 模擬套一次倍率。這是 Sixty Four 的獨立實作，不是直接套 Unity 補丁。`openspeedy-guard.exe` 是 x64 C# helper；僅支援 `speedpatch64.dll` SHA-256 `c72ba4d9e35c5c00efc113e541da91a02dbc0c85c6fb2effff5cee63898bc70c`，並驗證 PE export、`SP_IsEnabled` RVA `0x1170` 的機器碼，以及兩個 RIP 位址都指向對齊的 `pEnabled` slot `0xa458`，不符就拒絕。它保留原 `OpenSpeedy.<pid>` named mapping，將目標程序 slot 指向一頁私有 false 值，讓 native hook 不再縮放時間／等待；helper 自己載入 DLL 後也立即停用自己的原生 hook。

所有相同 exe 路徑的遊戲 main／renderer／utility 都須保護成功，才可送出啟用的 speed sample；實際模擬啟用條件是原 main／renderer 旗標的 OR，utility 旗標不啟用模擬。helper 自己新載入 DLL 會造成 `SP_Install` 預設啟用，因此 bootstrap 只停用自己新注入的 PID；使用者已先注入的既有旗標則保留。設定中存著 100 倍不代表啟用，新 bootstrap 的程序仍為 Off／有效 1 倍，必須由使用者明確打開遊戲的 OpenSpeedy 開關。退出 helper 不會把目標 slot 改回去或釋放仍被 hook 參照的 false page；要恢復原生掛鉤使用方式須重開遊戲。

`simulation-clock.cjs` 在 runtime 包裝 `game.updateLoop`，保留原 `updateCycle` 與原生遊戲規則；每次 worker tick 將真實 elapsed 乘有效倍率，以不超過 64 ms 的 substep 執行，最多 64 步、約每 4 真實 ms 在步驟邊界讓出，最後只送一次 worker acknowledgement。4 ms 是 cooperative budget，不能中斷單一步驟或保證硬上限。backlog 最多保留一真實秒對應的模擬時間；倍率改變會丟掉舊 backlog，renderer 距離最後收到狀態同步超過 2 秒即回到 1 倍。沒有全域覆寫 JavaScript clock。只縮放 dt 驅動的系統；render／native clocks、日曆、autosave 及依 performance 時間判斷的成就維持真實時間，不能宣稱所有遊戲功能完全等價加速。原生離散事件、每次 tap 的燃料成本及步長效應仍存在。

安裝以 `-EnableSpeedControl` 明確啟用此架構；六個 runtime 檔案為原 bridge 三檔加 `simulation-clock.cjs`、`speed-control.cjs`、`openspeedy-guard.exe`，不安裝 C# 原始碼。helper 和其餘 runtime hash 記錄在 `apiIsolation.files`，DLL hash 另外記錄；既有啟用設定升級時會保留，legacy 未啟用安裝則不自動啟用。真實套件 fixture 已驗證 DLL／helper 缺失拒絕、六檔寫入後失敗的完整回滾、port／認證及 native hashes 保留、升級與可重試卸除。此版已正常保存、關閉，再經 Steam Play 自動載入 bridge／gateway／helper；Node 全套 74／74、helper 隔離自測 21／21 通過。

實機先在「保存倍率 100、遊戲開關 Off」量到 simulation 約 0.996 倍、native clock 約 0.996 倍。On／100 倍兩個分開約 30 秒窗口，分別 98／91 組取樣零錯誤，simulation 約 99.992／100.212 倍，native clock 約 1.000／1.002 倍，render clock 約 1.000／1.002 倍，沒有 backlog 或 dropped simulation；這是兩段短期證據，不是永久穩定承諾。100→10 後約 9.979 倍，Off 後約 0.997 倍；實際 GUI 再關閉並保留設定 100 時，約 3 秒量到 1.002 倍。rollback recovery counter 僅啟動時為 1，之後倍率切換未增加。100 倍下兩次 `holdMs:1000` 成功釋放，總回應約 1159／1030 ms，外部等待約 1013／1008 ms，active 最後為 false。GUI 的 100 倍預設已選取並持久保存；仍須依遊戲開關決定實效倍率。

後續另在 100 倍連續觀察約 119.79 真實秒，599 次 state 與 599 次 health 全部零錯誤：credited simulation 約 100.175 倍，native clock 約 1.00173 倍，gateway wall clock 約 1.00005 倍。state 的 P95 約 173.66 ms、最慢約 277.36 ms，health 最慢約 16.95 ms；rollback recovery counter 維持 1、沒有新增倒退恢復。這補足當次場景約兩分鐘的持續運作證據，仍不是長期穩定、所有場景或實際產量完全 100 倍的保證，也未驗證目前隔離版本的 1000 倍。

# 歷史高倍率量測：不能外推到目前原生時鐘隔離版本

官方 OpenSpeedy UI 可設定 0.001～1000 倍，這是工具設定範圍。Sixty Four 1.2.1 沒有可據此宣稱的固定有效倍率上限：原生 worker 每 5 ms 嘗試送出一次 tick，等 acknowledgement 才再送；每次 simulation delta 上限 1000 ms，超過部分直接丟棄，沒有 backlog。worker 與 rAF 分開，但共用 renderer 執行緒。5 ms 是被調整的 timer 參數，不能直接推成真實 200 Hz 上限。

移除上述 main timeout 前的中間版，已用官方 CLI 設定並核對程序倍率，在 1／10／100／300／1000 倍各取短窗口；共 153 組 state／health 取樣沒有錯誤，health 最慢小於 15 ms，gateway wall clock 與外部時鐘比例約 1.001。首次降到 1 倍跨越時鐘倒退，該窗口的 clock／simulation ratio 無效，不能拿來當 1 倍效能。100／300／1000 倍窗口的 credited simulation 約為 93.4／283.1／618 倍，另一次較短 1000 倍窗口約 429 倍。這是有觀測開銷的短期遊玩時間統計，不是產量／深度倍率，也不是穩定或最高可達速度。1000 倍的 state 最慢約 1114 ms，health 仍快，顯示 API 排程隔離不能消除遊戲 IPC 延遲。不能只看 settings.json 的保存值當目前程序有效倍率。

高倍率下 renderer／main 最忙執行緒約用滿一顆核心的 98～100%；同時整張 GPU 的使用率快照約 35～39%，後者不是每遊戲的 GPU profile，不能推出 GPU 沒瓶頸或指認特定函式。尚未套用遊戲效能優化。

原生程式與精確方法 VM 已證明：Pump 大步灌入使 cube 超過 64 時，cube 仍填滿但該步可能不增加深度；Entropic 的 interval 300 ms 以 dt=1000 跑 10 秒只觸發 10 次，逐小步則接近 33 次；多種機器每次更新至多處理一次事件且重設 timer、捨棄 overshoot，Preheater 每次 tap 還有不依 dt 的燃料成本。因此移除 1000 ms cap、直接放大 dt 或任意改更新頻率都會改玩法，不是單純優化。VFX 的完成 callback 負責入帳與補料，不能為了省繪圖而停止 `updateVFX`。未來先量實際更新率、cap 次數、產量／深度及耗時，再評估僅減繪圖或服務排程的改動。

# 卡頓診斷：分開量音訊與 JavaScript 執行緒

不能把 renderer 程序合計約 1.28 顆核心當成 JavaScript 執行緒飽和；必須用 `GetThreadDescription` 配對逐執行緒 CPU。2026-10-31 同一場景的 100→10→100 倍短窗中，`AudioOutputDevice` 約占 0.998／1.000／0.998 顆核心，`CrRendererMain` 僅約 0.19～0.215 顆；降速未立即清除音訊高負載。原生 `game.js:2481–2524` 的 `playSound` 每個事件建立 3～4 個 AudioNode，`source.onended` 的 disconnect 整段被註解，僅以 `setTimeout` 降低 `stackSize`；一般上限 128，`forced` 可越過。這是後續 AudioContext／CPU stack 對照與清理修復候選，尚未證實洩漏或卡頓因果。WPR CPU profile 在開始錄製前即被 Windows profiling policy 拒絕，因此未取得 native CPU stack。

上述比較未重啟或更動建築：health median 均約 0.8 ms，state median 約 1498／804／1096 ms，首次 100 倍另有一次 8 秒逾時；10 倍與第二次 100 倍的實測模擬速率約 9.626／35.716 倍。首次 100 倍的 `renderDelta` 曾達 1517～1616 ms，恢復 foreground 後仍有長 frame；起初 `renderLast` 不變也可能與背景狀態有關，這次未以畫面觀察確認。native clock 約 1 倍、rollback 增量為 0，原生時鐘隔離仍 active，不能把 CPU 飽和當成唯一根因，也不能以 health 快代表畫面正常。這是動態世界的短窗對照，並非同存檔重播；結束已核對恢復 enabled／100 倍且 isolated。
