---
title: Sixty Four 的 Electron agent API 接入與原生操作語意
scope: project
project: sixtyfour
status: active
updated: 2026-10-31
---

## 原生操作與存檔

- **找不到 `window.game` 或 bridge 在 Steam Play 後失效。** 1.2.1 renderer 的 `let game` 是 lexical。正常 Steam 走 `main.js → game/index.html → scripts/game.js`；Steam 更新可能覆蓋本地 hook。安裝前備份原檔與 profile，核版本/hash，啟動後讀 runtime marker，再做 state/catalog/resume 實際操作。部分寫入要完整回滾，卸載重試接受已恢復的原始 hash。
- **放置成功卻跳過成本／解鎖，或回收後資源沒立刻增加。** 原生用 `pickupItem`、hover 及 `processClick()`。Adapter 額外驗解鎖、唯一性、plane、成本、格位、升級目標和實物；catalog availability 必須與 place 的限制一致。回收先驗 `canEraseNow()`、選原生 `eraserType`，保留最後 excavator 等保護，讀 `getRealPrice(name, true)` 並等待 VFX 入帳。退款倍率 1 可回基礎價格，不能硬套折扣。
- **Pump 長按中途取消。** 原生指標更新會蓋過 hover；每次目標 native update 前設定 hoveredEntity，`finally` 還原原 own/prototype 狀態並釋放。連點每次查可互動性；cube 破即停。Menu 顯示時拒絕世界操作，用 resume。
- **Save 回覆成功後準備重開或驗 Steam Cloud。** `abstractv03<Steam-id>` localStorage 保存與 IPC cloud request 只能各自確認。官方選單退出先 `saveGame()` 再 IPC quit，main 自己沒有補存。遇 `preventSaving`／`preventCloud` 如實報結果。保留 profile 備份。

## 時鐘、回覆與延遲

- **背景連點卡住或 504 後又有操作。** 外部 standalone Node 24+ gateway 須不載入 OpenSpeedy，使用 `process.hrtime.bigint()` 算真實時間。HTTP loopback/認證/動作白名單，佇列容量 16、gateway deadline 8 秒。Begin/end hold 經私用 IPC 配對；504 代表客戶端等待到期，原生 RPC 加清理到 acknowledgement 前不得推進佇列。別在受加速 main 增添 timer/race；單獨記 requested hold、外部等待與進出 round trip。
- **反覆呼叫健康檢查有約 14 ms 固定延遲。** Node 24.19.0 內附 Undici 7.29.0 的 idle socket validation timer 是實測成因；server work 約 0.017 ms。以 `client.mjs` 重用原生 HTTP keep-alive pool，或 `node <package-root>/sixtyfour.mjs session` 保留 stdin 並用帶 id 的 NDJSON；勿自動重送 mutation。診斷見 [Node HTTP 延遲](../../tools/node/http-client-latency.md)。
- **模擬和畫面一方停住，另一方仍活著。** 兩個都有真實故障：舊 100→10 rollback 可讓 worker 停但 rAF 活著，另有 renderer/rAF 停而 sim／API 前進。用實體 canvas、render timestamp、simulation 與外部 clock 同時驗。舊 rollback 防護從仍更新的 render loop 重建 clock worker；現行 `-EnableSpeedControl` 隔離實際原生時鐘，只依有效 OpenSpeedy enable flag 縮放遊戲 dt。新 PID 設定值 1000 且 Off 仍為 1×；超過 2 秒無樣本回 1×。逐步模擬有 backlog／cooperative 限制，autosave、日曆、render 和 audio 時間仍走真實時間。
- **高倍數快速跑但卡頓被全歸給聲音。** 同程序交錯 native/bounded/silent 及復測，長時段才分得出 renderer/JS、AudioOutputDevice、server outlier。100× 實測 bounded 降低本場景 audio CPU，state 延遲不隨靜音單調下降；未證長期穩定。換 DLL/遊戲版本先核隔離支援和實機 marker，再驗 sim、audio、render 與 save。
- **看到歷史 `debugEval` 自訂診斷入口。** 正式 API 保持固定白名單且不接受任意 JavaScript。若工作樹留有舊實驗，停用並重啟該程序，再從正式介面核對拒絕和相同功能讀回；改設定不會即時替換已啟動程序的配置。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/sixtyfour/electron-agent-api.md)
