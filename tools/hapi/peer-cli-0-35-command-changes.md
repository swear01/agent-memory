---
title: HAPI 0.35 peer 指令對照：哪些 CLI 還在、archive 怎麼做
scope: tools/hapi
tool: hapi
status: active
confidence: high
created: 2026-11-18
updated: 2026-11-19
tags: [hapi, spawn-peer, ping-peer, inspect-peer, archive-peer, stop-peer, wait-peer, abort-peer, delete-peer, unknown-command, skills]
---

# HAPI 0.35 peer 指令對照

2026-11-18 在 Mac `0.35.0.1` 逐一跑 `hapi <cmd> --help` 確認。

| 0.30 維護版 | 0.35 |
|---|---|
| `hapi spawn-peer` | 還在，只是 `hapi --help` 沒列。沒有 `--remit-id`；`--permission-mode` 省略時是 yolo，依 [session 許可模式](session-permission-preference.md) 明確傳。在 session 內優先用 MCP `spawn_peer`。 |
| `hapi ping-peer` | 還在。inactive session 會被拒（exit 5），加 `--wake` 才會 resume；有 `--delivery-mode auto/queue/steer`。 |
| `hapi inspect-peer`、`hapi set-mode` | 還在。 |
| `hapi wait-peer` | Unknown command。在 session 內用 MCP `spawn_peer`／`ping_peer` 帶 `waitForReply: true`，然後結束這一輪；在 shell 裡輪詢 `hapi inspect-peer <id>`（0.35.0.1 沒有 `--json`，會回 `unexpected flag: --json`；要結構化資料就打 `GET /api/sessions/:id`）。 |
| `hapi archive-peer` | Unknown command。用 `hapi-session-control` skill 的 `scripts/archive-peer.py <full-id> ...`（呼叫 Hub API `POST /api/sessions/<id>/archive` 並讀回 `active=false`）。只接受 active session，inactive 回 `Session is inactive`。 |
| `hapi delete-peer` | Unknown command。inactive 紀錄用 `DELETE /api/sessions/<id>`。 |
| `hapi stop-peer` | 沒有替代品。runner `stop-session` 會順便 archive，不是 process-only stop。 |
| `hapi abort-peer` | 沒有 CLI 替代品，從 Web UI 中斷這一輪。 |

archive 之後讀回 lifecycle 已為 archived；`active=false` 也可能是 runner 斷線，不能單獨當成已封存。見 [runner 程序查證](supervisor-safe-operations.md)。

## 舊的 bundled skill 與 Skillshare

`hapi-session-control` 改由 Skillshare 發布。v0.35.0.1 還包含 fork 的舊安裝器，啟動 agent 時可能把帶 `.hapi-managed` 的實體目錄更新成 bundled 版本；指令表或 archive script 不一定等於 Skillshare 版本。遇到工具引用已淘汰的 `archive-peer`，先核對該 agent 實際載入的 skill path／symlink 和 release，再按 HAPI 維護手冊移走確認屬舊安裝器的副本。不要覆寫本機自管 skill。

把過去 Windows 的 target exclude 或暫未遷移狀況當成當批證據。現行 target 與機群範圍依 HAPI 手冊核對。原始部署歷史和版本對照見 [整理前紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/peer-cli-0-35-command-changes.md)。

# Agent 自己 archive（archive-peer.py，2026-11-18 實測）

- 認證用環境變數 `CLI_API_TOKEN`（runner 從 `runner.env` 繼承給子程序）；Mac 的 `~/.hapi/settings.json` `cliApiToken` 對公開 hub 回 `Invalid access token`，只當備援。
- 公開 hub 在 Cloudflare 後面：Python 預設 User-Agent 會被擋（403 `error code: 1010`），要送自訂 UA。
- 已用一次性 Claude session 驗證：archive 後 `inspect-peer` 讀回 `lifecycle: archived`，再跑一次回 `already_inactive`，不存在的 id 回 `not_found`；測試 session 已 `DELETE /api/sessions/<id>`。
