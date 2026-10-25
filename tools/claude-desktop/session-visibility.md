---
title: Claude 桌面版 Code 分頁看得到哪些 session（HAPI session 為何看不到）
scope: tools/claude-desktop
tool: Claude desktop app
status: active
confidence: medium
evidence: 2026-10-25 查 code.claude.com/docs/en/desktop 與 /remote-control；HAPI 啟動參數查 cli/src/claude/sdk/query.ts。桌面側欄實際行為未在 app 內實測。
created: 2026-10-25
updated: 2026-10-25
tags:
  - claude-desktop
  - remote-control
  - hapi
  - child-session
---

# Claude 桌面版 Code 分頁看得到哪些 session

## 結論

- 桌面版 Code 分頁只管它自己跑的 session：本機、SSH、WSL（文件："Claude sees only the sessions the desktop app runs itself"），加上 cloud session。終端 CLI、VS Code、Agent SDK 起的 session 不會列進來。Claude Code 的 transcript 存在各機器自己的 `~/.claude/projects/<cwd>/<id>.jsonl`，沒有帳號層級的統一清單。
- 所以 HAPI 在遠端 Linux 機器上的 Claude session 不會出現在桌面版。HAPI 從 web 開的是 remote mode：`claude --print --input-format stream-json --output-format stream-json`（headless SDK 用法）。

## 想在 Claude 官方介面看到的路

- Remote Control：`/rc`、`claude --remote-control`，或 user `~/.claude/settings.json` 設 `remoteControlAtStartup: true`。session 會出現在 claude.ai/code 與 Claude app。需要 claude.ai 訂閱登入（API key 不行）；`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 或 `DISABLE_GROWTHBOOK` 會讓它不可用。自動連線只對**互動式**行程生效，所以大概只有 HAPI local mode（TUI）會連上；`--print` 的 remote mode 未驗證。
- 桌面版 SSH session：是桌面版自己在遠端另開的新 session，不是 HAPI 的 session。
- `/desktop`、`claude --desktop --resume <id>`：只支援本機 macOS 與 x64 Windows，遠端 Linux 用不到。

## Child session

- 桌面版**有** parent/child：由 session 啟動的 session（例如 suggested task 的 **Start**）會在側欄列在「目前 session 底下」；session 管理工具裡看得到 `parentSessionId`、`startedBy`、`detached`（見 [child-session-auto-archive](child-session-auto-archive.md)）。
- Claude Code 的 subagent（Agent/Task 工具）不是獨立 session，只顯示在對話裡的 Agent 卡片。
- HAPI（fork main 與 upstream，2026-10-25）沒有 session 樹：`spawn_peer` 開出的 session 和 parent 平行列在「目錄 · 機器」群組下，metadata 不記 parent。最接近的是 tiann/hapi#1922（heavygee，open），只在 remit 訊息裡蓋一個 Parent 連結。
- 決定（2026-10-25，使用者）：HAPI 不做 parent/child session 樹。HAPI 對應的是 Claude Code CLI，CLI 本身沒有 child session，只有桌面版有；做了太激進。除非 Claude Code CLI 加上這個功能，否則不要再提。
