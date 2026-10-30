---
title: Claude 桌面版只讓 child session 在 PR 合併後自動歸檔
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
evidence: 2026-10-25 用 ccd_settings / ccd_pr / ccd_session_mgmt 工具查證設定與欄位；mod 通過 `claude plugin validate`、`tsc` 與 `claude plugin test`。2026-10-31 從 app log 查證 auto 模式下 hook 的 set_monitor 全被擋（bypass 4/4 成功、auto 0/8）。0.1.2/0.1.3 的「由 agent 呼叫」做法尚未在真的 auto 模式 child PR 上端到端驗證。
created: 2026-10-25
updated: 2026-10-31
tags:
  - claude-desktop
  - auto-archive
  - plugin
---

# 只讓 child session 在 PR 合併後自動歸檔

## 需求與限制

- 全域設定 `auto_archive_on_pr_close`（Settings → Claude Code，帳號層級）只有開／關，不能加條件；開著會把所有 PR 合併或關閉的 session 都歸檔。
- 這個開關在桌面 app 裡，不在 Claude Code engine 裡；mod（plugin hooks）攔不到它。
- 但每個 session 還有自己的 per-PR 開關：`mcp__ccd_pr__get_status` 回傳 `monitor.auto_archive_on_close`，`mcp__ccd_pr__set_monitor({ auto_archive_on_close: true })` 可以打開。只對已綁定、仍開著的 PR 有效；PR 合併**或關閉**都會歸檔。全域開關關掉時，這個值預設是 false。
- 判斷 child：`mcp__ccd_session_mgmt__get_session("self")` 有 `parentSessionId` 且 `detached` 不是 true（`list_sessions` 列裡則是 `startedBy`）。

## 做法（已部署在 Mac）

- 全域 `auto_archive_on_pr_close=false`，`auto_archive_inactive_days=4`。
- 本地 plugin `child-auto-archive@local`：原始碼在 transfer_MAC 的 `stow/agents/.agents/claude-plugins/mods/child-auto-archive`，同時記在 `plugins.json`。在 `session.start` 和每個 `turn.complete` 之後（0.1.1 起），用 `$.mcp.call('ccd_session_mgmt'|'ccd_pr', ...)` 檢查；是 child 且已綁 PR 就呼叫 `set_monitor` 打開開關，真正的歸檔交給 app。main session 一律不動。
- 只對之後新開或重新開啟的 session 生效；開關一旦打開就記在 app 裡，session 沒在跑時 PR 合併也會歸檔。
- **auto 模式**（0.1.2 起）：app 會擋掉 mod 自己的 `$.mcp.call('ccd_pr','set_monitor')`，因為對話裡沒有任何要求（同 anthropics/claude-code#99214：`classifier gave no verdict ... the request that produced this action did not ask for one`；加 `permissions.allow` 沒用）。所以 mod 改用 `prompt.compose` 只在 child 的 system prompt 加一段，請 agent 開 PR 後自己呼叫 `set_monitor`；hook 自己的呼叫只在 bypass 模式有效，被拒一次後就 toast 一次並停止重試，之後只盯 `get_status` 等開關打開。
- 查法：app log（`~/Library/Logs/Claude/main*.log`）裡成功的話會有 `Session <id> auto-archive on PR close set to true`；spawn 行的 `permissionMode ... effective=` 可看該 session 的模式。

## 走過的彎路

- 0.1.0 在 `gh pr create` / `git push` 的 `tool.call` hook 裡檢查：app 是在讀到工具輸出之後才綁 PR（log：`[GitHubPrManager] Bound github #N ... ahead of its first read`），而這時 hook 已經跑完了，所以第一次檢查幾乎一定看到 `bound:false`。只有之後又 push 過一次的 session 才會成功。Verto PR #31 的 child 只 push 一次就合併了，所以沒被歸檔；而 PR 合併後 `set_monitor` 會拒絕，之後補救不了。2026-10-26 改成每個 turn 結束時檢查（transfer_MAC PR #68）。
- 不要快取「是不是 child」：session 隨時可能被 detach。PR 合併後也不要停止檢查：同一個 session 之後可能再綁新的 PR。

- 每小時用 scheduled task 掃 `list_sessions` 再 `archive_session`：被 auto mode classifier 擋下，而且是輪詢，不如上面的事件式做法。
- 在 mod 裡聽 `session.receive` 的 GitHub merged 事件、再 `archive_session("self")`：需要 session 活著、會跳批准卡片；改用 per-PR 開關比較簡單。
- `claude plugin validate` 要求傳 `$` 的 helper 是**檔案頂層**的 function 宣告，不能是 `register` 裡的 closure；所以狀態只能放模組層級、在 `register` 裡重設。Gemini review 會建議「把狀態搬進 register」，照做的話 module 會載入失敗（transfer_MAC PR #75 試過）。
- 0.1.1 把 `get_session` 失敗當成「不是 child」並永久快取；0.1.2 又快取了 `isChild=true`，違反上面「會被 detach」那條。0.1.3 只快取「不是 child」（session 不會後來才有 parent）。
- `~/.claude/dev-mods/<session-id>/` 只給當前 session 熱載入；要讓所有 session 載入，得用 folder marketplace + `claude plugin install <mod>@<marketplace> --scope user`。
