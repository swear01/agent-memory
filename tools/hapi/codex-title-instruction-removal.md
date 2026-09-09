---
title: HAPI Codex 原生標題同步與自動命名
scope: tools/hapi
status: verified
updated: 2026-09-09
---

調查 Codex 新會話不自動命名時，先比對實際部署 tag，不能直接相信本機舊 checkout。

已透過 GitHub tag 原始碼確認：swear01/hapi v0.29.0.5 的 `cli/src/codex/utils/appServerConfig.ts` 會呼叫 `getCodexSystemPrompt()` 並注入 HAPI 標題指令。v0.29.0.6 的 `resolveInstructions()` 僅轉交 caller 提供的指令；發版提交 `92e025f26460815da6de6b4397010c7d8ebab6d3` 移除 `cli/src/codex/utils/systemPrompt.ts` 和注入邏輯。前後版本字串斷言驗證通過。

Zeus 實際 HAPI 0.29.0.6 / Codex 0.153.0 新會話的 rollout 沒有標題 developer instruction；HAPI MCP startup 為 ready。此證據支持 HAPI 指令移除導致不再主動命名，尚未證實 Codex 事件格式有回歸。部署版仍保留 change_title bridge 與事件處理，不能把「本回合工具清單沒顯示」當成工具已被刪除。

後續原生路徑調查修正：不能僅憑標題指令移除就判定完整根因，也不應直接建議恢復舊指令。Codex rust-v0.153.0 的 `codex-rs/tui/src/app/thread_title.rs` 在 TUI 建立 temporary thread 並執行 structured turn 產生名稱；`app/event_dispatch.rs` 的 Automatic 分支再呼叫 `thread_set_name`。HAPI remote app-server 路徑不等於啟動 TUI。官方 app-server 支援 `thread.name` 與 `thread/name/updated`，但部署 v0.29.0.6 的 converter 和 launcher 沒有處理該通知；字串斷言已確認。

Zeus config 的 `sqlite_home` 指向 host-local `/var/tmp/<runner-user>-codex-sqlite`，不要查舊的預設 Codex home DB。實際新會話的 `threads.name` 為 NULL，`title` 是初始訊息文字。故這次不只是 HAPI 漏顯示一個已產生的原生名稱；也缺少原生自動命名的觸發路徑。尚未定位哪個 Codex 版本開始有此 TUI 行為，不宣稱 Codex 更新造成協定回歸。

Hub titleSuggestion 是另一條手動 Generate 流程，與這次 Codex 原生自動命名分開。此工作未部署正式 fleet。

PR 歸屬：上游 `tiann/hapi#1771`（調查起點 head `2315387c80780047f59c72f868bf4af886a09981`）移除指令，2026-09-09 查詢仍 open；fork 發版 `swear01/hapi#17` 已 merged。原需求 `tiann/hapi#1769` 明確將 title generation 列為應移入 HAPI code 的產品自動化。修正建議應補原 PR 的替代路徑與回歸驗證，不能只保留「沒有 prompt」測試。既有 `createNativeSessionTitleMetadataSync` 可重用；Codex API 原生名稱同步與無名稱時的獨立生成是兩件事。

修正已在 #1771 工作分支實作並完成 Codex 0.153.0 隔離環境實測：重用 `createNativeSessionTitleMetadataSync`；remote 接收 `thread/name/updated` 和 start/resume/fork 回傳名稱，local 每五秒 `thread/read`。remote 第一個完成的 task 若無名稱，使用獨立 ephemeral、read-only、禁用工具/MCP/plugins/hooks 的 structured request 生成名稱，透過 `thread/name/set` 寫回，再同步 metadata。實測 `thread/read.name` 與 metadata summary 一致；工作對話仍沒有 HAPI 標題 prompt，manual metadata.name 優先。

跨 app-server 程序讀取剛建立但尚未持久化的 thread 會出現 `thread not loaded`。remote 啟動時應用 start/resume 回傳的 name，而非立即另開 client readThread；真實整合驗證須先完成一次主要 thread turn，再以另一個 app-server 讀取。生成 client 完成後 disconnect，local 輪詢 client 在 session stop 時關閉，避免額外程序累積。deadline 必須涵蓋 initialize/config/model/read/start/turn/set 全流程，不能只限制輸出等待。
