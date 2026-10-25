---
title: Claude 桌面版只讓 child session 在 PR 合併後自動歸檔
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
evidence: 2026-10-25 用 ccd_settings / ccd_pr / ccd_session_mgmt 工具查證設定與欄位；mod 通過 `claude plugin validate`、`tsc` 與 4 個 `claude plugin test`。尚未在真的 child PR 合併上端到端驗證。
created: 2026-10-25
updated: 2026-10-26
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

## 走過的彎路

- 0.1.0 在 `gh pr create` / `git push` 的 `tool.call` hook 裡檢查：app 是在讀到工具輸出之後才綁 PR（log：`[GitHubPrManager] Bound github #N ... ahead of its first read`），而這時 hook 已經跑完了，所以第一次檢查幾乎一定看到 `bound:false`。只有之後又 push 過一次的 session 才會成功。Verto PR #31 的 child 只 push 一次就合併了，所以沒被歸檔；而 PR 合併後 `set_monitor` 會拒絕，之後補救不了。2026-10-26 改成每個 turn 結束時檢查（transfer_MAC PR #68）。
- 不要快取「是不是 child」：session 隨時可能被 detach。PR 合併後也不要停止檢查：同一個 session 之後可能再綁新的 PR。

- 每小時用 scheduled task 掃 `list_sessions` 再 `archive_session`：被 auto mode classifier 擋下，而且是輪詢，不如上面的事件式做法。
- 在 mod 裡聽 `session.receive` 的 GitHub merged 事件、再 `archive_session("self")`：需要 session 活著、會跳批准卡片；改用 per-PR 開關比較簡單。
- `claude plugin validate` 要求傳 `$` 的 helper 是**檔案頂層**的 function 宣告，不能是 `register` 裡的 closure。
- `~/.claude/dev-mods/<session-id>/` 只給當前 session 熱載入；要讓所有 session 載入，得用 folder marketplace + `claude plugin install <mod>@<marketplace> --scope user`。
