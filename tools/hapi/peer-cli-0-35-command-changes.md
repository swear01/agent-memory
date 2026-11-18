---
title: HAPI 0.35 peer 指令對照：哪些 CLI 還在、archive 怎麼做
scope: tools/hapi
tool: hapi
status: active
confidence: high
created: 2026-11-18
updated: 2026-11-18
tags: [hapi, spawn-peer, ping-peer, inspect-peer, archive-peer, stop-peer, wait-peer, abort-peer, delete-peer, unknown-command, skills]
---

# HAPI 0.35 peer 指令對照

2026-11-18 在 Mac `0.35.0.1` 逐一跑 `hapi <cmd> --help` 確認。

| 0.30 維護版 | 0.35 |
|---|---|
| `hapi spawn-peer` | 還在，只是 `hapi --help` 沒列。沒有 `--remit-id`；`--permission-mode` 省略時是 yolo，依 [[session-permission-preference]] 明確傳。在 session 內優先用 MCP `spawn_peer`。 |
| `hapi ping-peer` | 還在。inactive session 會被拒（exit 5），加 `--wake` 才會 resume；有 `--delivery-mode auto/queue/steer`。 |
| `hapi inspect-peer`、`hapi set-mode` | 還在。 |
| `hapi wait-peer` | Unknown command。在 session 內用 MCP `spawn_peer`／`ping_peer` 帶 `waitForReply: true`，然後結束這一輪；在 shell 裡輪詢 `hapi inspect-peer <id> --json`。 |
| `hapi archive-peer` | Unknown command。改用 Hub API `POST /api/sessions/<id>/archive`（JWT 取法見 [[session-permission-preference]]）。只接受 active session，inactive 回 `Session is inactive`。 |
| `hapi delete-peer` | Unknown command。inactive 紀錄用 `DELETE /api/sessions/<id>`。 |
| `hapi stop-peer` | 沒有替代品。runner `stop-session` 會順便 archive，不是 process-only stop。 |
| `hapi abort-peer` | 沒有 CLI 替代品，從 Web UI 中斷這一輪。 |

archive 之後一定要用 `inspect-peer` 讀回 `active=false`，才能回報「已收掉」。

# 為什麼 agent 還會打舊指令（2026-11-18）

一個 session 要收掉自己開的 Grok peer，照舊指令跑 `archive-peer`，先回報收掉了，之後才發現指令不存在、peer 還是 active。舊指令有兩個來源：

1. **HAPI 自己管理的舊 skill 副本**（目錄帶 `.hapi-managed`）。0.30 維護版的安裝器寫進各 Agent 的技能目錄；0.35 只更新 Claude、Codex、Cursor、agy 的 `hapi-session-control`，不會更新 Grok（`~/.grok/skills`）、Pi（`~/.pi/agent/skills`）的副本，也不會刪 `hapi-session-runtime`。Grok 會同時讀 `~/.grok/skills` 和 `~/.claude/skills`，同名時 `~/.grok/skills` 的舊副本蓋過新版。2026-11-18 已在 Mac 把這些舊副本移走（Pi 的換成 0.35 版）；其他機器還沒查。
2. **記憶裡還在教舊指令的筆記。** 同日已改寫成上表的做法；歷史紀錄保留，但標明只適用 0.30.x。

檢查方式：

```bash
find ~ -maxdepth 5 -type d \( -name hapi-session-runtime -o -name hapi-session-control \) -path '*skills*' -not -path '*/.hapi/runtime/*' -not -path '*/.agent-worktrees/*'
```

對每個結果 grep `archive-peer|stop-peer|wait-peer`。有命中的 `hapi-session-control` 用 `<HAPI_HOME>/runtime/<version>/skills/hapi-session-control/SKILL.md` 取代；`hapi-session-runtime` 直接移出技能目錄。Grok 不要加 Skillshare target，原因見 `projects/transfer_MAC/skillshare.md`。
