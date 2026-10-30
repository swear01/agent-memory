---
title: Node.js 本機 HTTP 延遲要拆出 idle socket 排程與 CLI 首回應耗時
scope: tools/node
tool: nodejs
status: active
updated: 2026-10-30
---

# 先定位送出 request 前的等待

Windows／Node.js 24.19.0（內建 Undici 7.29.0）曾對同一本機 health endpoint 反覆量到 fetch 約 15 ms、`node:http` keep-alive 小於 1 ms。相同程序交錯各 30 次的 median 為 15.129／0.379 ms，皆零錯誤；不能只看端到端時間就歸因於 server、網路、TCP handshake 或遊戲 IPC。

另以相同 Node fork 出獨立 server 的 fixture，交錯各 40 次並追蹤 `undici:request:create`、`undici:client:sendHeaders`、server 收送與 idle-validation timer。fetch 40 次全部經過 `scheduleIdleSocketValidation`：timer 實際等待 median 13.6747 ms，request create 到 send headers 13.8396 ms，send 到 server 僅 0.2315 ms、server work 0.0167 ms。fetch 總時間 median 14.7763 ms，原生 HTTP 0.4819 ms；fetch 全程重用同一個 local port、沒有新連線。該輪證據把主要額外等待定位到送出 headers 前的 idle socket validation 排程。

安裝中的 built-in source 與 [Node v24.19.0 隨附的 Undici 原始碼](https://github.com/nodejs/node/blob/v24.19.0/deps/undici/undici.js) 經 CRLF 正規化後逐字相同。`scheduleIdleSocketValidation` 使用 `setTimeout(callback, 0)`，`resumeH1` 在 validation pending 時暫停重用 idle socket，callback 才恢復派送；這和 trace 的等待位置相符。要核對 Node 自己打包的 source，不能僅依 `process.versions.undici` 找同名 upstream tag，兩者可能有不同補丁。

這不證明 Windows kernel 的特定 timer 成因，也不是所有 fetch／Node／Windows 都固定慢 16 ms。另一程序曾量到約 1 ms；後續交錯 default fetch、明確 Connection keep-alive、帶 AbortSignal 各 30 次，median 仍約 15.64／15.51／15.47 ms，故 header 變更不是可靠修復。保留版本與呼叫條件，升級後重新量測。

# 專用低延遲 client 與持續執行的 CLI

對已確認路由的小型 agent API，可用標準庫 `node:http`／`node:https` 與明確擁有的 keep-alive Agent，避開上述 client 路徑。整個 agent run 重用同一 client，退出時關閉 pool 與 outstanding requests；不改全域 timer、不 patch Undici 內部。較早一次性 CLI 的完整耗時約 100～127 ms，包含程序初始化、設定、HTTP 與輸出，不能把整段都算成純程序啟動。以單一長駐 Node 程序讀 NDJSON 命令，可避免每筆指令重做程序初始化，並重用 HTTP pool。

傳輸替換要保留精確路由／method 與認證限制、回應大小上限、涵蓋 socket queue／response body 的完整 wall-clock deadline，以及退出清理。Socket inactivity timeout 不能單獨代表完整 deadline。對會改狀態的操作不要自動 retry：timeout／斷線後，server 可能已經完成操作；先查狀態再決定後續動作。對外部工具或 agent 的整合範例應把 SDK/client 建在迴圈外。

測試至少包含相同 endpoint 的交錯 A/B、warm-up／連線重用、socket queue 與 body 延遲、半截回應、超大回應、close 取消與 mutation 不重送。端到端含 IPC／遊戲負載，不能把不同場景的 state 窗口當確定性重播；health 的同程序比較較適合隔離 client 成本。

最終專案 fixture／整合測試共 95 項通過、無 skip；持續執行 SDK 的唯讀實測，health 30 次 median／P95／max 約 0.470／0.604／0.732 ms，state 30 次 median 約 4.803 ms、最慢 179.450 ms，扣除每筆 server elapsed 後的 client residual median 約 0.839 ms。state 的遊戲／server 負載仍會產生 outlier；這些是該版本與當次窗口的結果，非一般效能保證。

# Windows stdout backpressure 也可能堵住終止流程

同一 Windows／Node.js 24.19.0 環境，以父程序停止讀取的 stdout pipe 和 2 MiB 回應重現：`process.stdout.write()` 可阻塞 JavaScript 執行緒；改用非同步 `fs.write()` 可讓 JavaScript 的 signal handler 繼續執行，但 `process.exit()` 仍可能等待堵住 pipe 的 worker，不能據此承諾立即退出。專用 CLI 採 EOF 正常排空；明確 SIGINT／SIGTERM 的 handler 先同步關閉 HTTP client，再只終止自身 PID（Windows `process.kill(process.pid, 'SIGKILL')`，該環境 exit code 為 1；Unix 為 130／143）。2 MiB backpressure fixture 對兩個 handler 均通過；測試直接觸發已註冊 handler，不代表所有 Windows console／signal 傳遞方式都相同。這是允許放棄待輸出結果的終止策略，不應套到需要完成持久化的程序。
