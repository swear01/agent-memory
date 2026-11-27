---
title: HAPI × Antigravity (agy) transport、model effort 與登入排查
scope: tools/antigravity
tool: Antigravity CLI (agy)
status: active
confidence: high
created: 2026-09-20
updated: 2026-11-18
tags: [antigravity, agy, hapi, integration]
---

## 先確認版本、登入與模型

- HAPI 的 agy flavor 是 remote mode，沒有本機 PTY 圖形介面。上游 #1320 最初包 TUI，#1591 改 headless print mode。舊版每輪 spawn，後續常駐 stdin transport 見 #2248。應按使用中的 release 核對對應架構。
- 模型清單先用 `agy --output-format=json models` 查。新登入可用 `agy -p "reply only with OK"` 實測；`Please sign in to view available models` 是登入訊號。單條 `You are not logged into Antigravity` 可能只是啟動競態。
- 登入只影響共用同一個 home 的 runner。憑證位於 `~/.gemini/antigravity-cli`，不要讀出內容。HAPI runner 只顯示 `[agy-headless] Starting headless driver`、探測後 cleanup，hub 的 session 沒有 message，需分別查登入、模型探測與實際推論。OAuth 詳見 [agy 登入排查](agy-consumer-oauth-login.md)。
- `agy -p /model --model gemini-4-argon-medium --effort high --output-format stream-json` 在 agy 1.4.10 實測回 `--model gemini-4-argon-medium conflicts with --effort=high`。讀 result 的 `error`，失敗時 `response` 可能為空。使用 base model `gemini-4-argon` 加 effort 可解析成 `gemini-4-argon-high`。用 `agy -p /effort --output-format json` 讀 `command.data.available`，不要照通用 help 猜該模型支援值。
- HAPI #2244 將設定正規化成 base model + effort。固定版本清除 stale effort，自訂模型名不盲拆字尾，僅對至少兩個 catalog effort variants 的 family 合併選項。Config RPC 返回實際套用的 model/effort，並保留 enqueue-time snapshot。動態 catalog 選項不可拿靜態 allowlist 保存或回讀。
- 初始化要先回報 bootstrap 與安裝 lifecycle/kill，再等 catalog。冷 cache 探測與非 Codex runner 的 startup deadline 同為 15 秒，順序顛倒會先殺 wrapper。早送訊息留在 `ApiSessionClient.pendingMessages`，取消／重送先和該 buffer 協調，避免重複執行或漏取消。

## 常駐 stream-json transport

HAPI #2248 驗證的是 PR 分支與 agy 1.4.11。截至當時讀回仍 open，部署須查實際 release。原生與 source driver 都完成兩輪同程序，第二輪保留 context，正常關 stdin 可退出。未以真實推論驗證 active Stop。

- `agy --input-format stream-json --output-format stream-json` 接收逐行 `{event:"user",message:{content:prompt}}`。同 model/effort/permission 重用程序，init 採用的 UUID 不屬於 config key。
- `result` 結束 turn，process close 結束 transport。先 flush planner/sendChain、派 acknowledgement，再接下一輪。stdout/listeners 隸屬 process，各輪 acceptance/ack/model attribution 分開；舊 process 晚到的 close/error 不應清掉新回合。stderr soft-deny 可能在 result 之後，以 idle callback 接住去重，不阻塞下一輪。
- 設定變更先關 stdin 給 500ms EOF 退場，必要時終止 process tree。Stop 終止舊程序，下輪用 `--conversation` 恢復。收到 prose、tool、user_input 或 result 即表示 prompt 已接收；只有尚未接收才做有限重試。kill 與 queue wakeup 相撞時，dequeue 項目須在 kill 返回前 restore/ack。
- 空白輸入在原生 CLI 會 ERROR+exit1，寫入前先攔截並保全訊息。`/model` 等原生自答命令會 ERROR+exit2。無 `control_request`／`control_response`，不能靠串流切換設定。result 的 `num_turns`／usage／duration 是 session 累計值。

## MCP 與 ACP

- 原生 `agy mcp add/remove/list/enable/disable` 管理 stdio／HTTP。CLI 使用 `~/.gemini/config/mcp_config.json` 或 workspace 的 `.agents/mcp_config.json`。IDE 使用另一份 `~/.gemini/antigravity/mcp_config.json`。看到 `No MCP servers configured` 先查 CLI 的實際檔案。
- HAPI 不注入自己的 MCP bridge，agy session 內的工具不會憑 HAPI skill 文件自動出現。能用的 CLI 路徑見 [HAPI 0.35 指令差異](../hapi/peer-cli-0-35-command-changes.md)。遠端不能用 TUI 的 `/mcp`，須預先設定 server 與 agy allow rule。
- agy 的權限由原生 allow/deny `settings.json` 與 CLI mode 控制。沒有 allow rule 時可能自動拒絕；不能從其他 HAPI flavor 的 hook 行為推論 agy。
- 2026-09 查核時，agy CLI 無 ACP 模式，但 Google 另有 registry `antigravity-acp` 與 `agy_acp_server.par`。該版本 HAPI 未接獨立 server，ACP early RC 觀察不能代替對現行 release 的查核。

版本對照、舊 fork audit、hub／runner 查詢和 PR 驗證可由 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/antigravity/hapi-agy-integration-status.md` 追溯。
