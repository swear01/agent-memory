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
| `hapi archive-peer` | Unknown command。用 `hapi-session-control` skill 的 `scripts/archive-peer.py <full-id> ...`（呼叫 Hub API `POST /api/sessions/<id>/archive` 並讀回 `active=false`）。只接受 active session，inactive 回 `Session is inactive`。 |
| `hapi delete-peer` | Unknown command。inactive 紀錄用 `DELETE /api/sessions/<id>`。 |
| `hapi stop-peer` | 沒有替代品。runner `stop-session` 會順便 archive，不是 process-only stop。 |
| `hapi abort-peer` | 沒有 CLI 替代品，從 Web UI 中斷這一輪。 |

archive 之後一定要用 `inspect-peer` 讀回 `active=false`，才能回報「已收掉」。

# 為什麼 agent 還會打舊指令（2026-11-18）

一個 session 要收掉自己開的 Grok（Cursor flavor）peer，照舊指令跑 `archive-peer`，先回報收掉了，之後才發現指令不存在、peer 還是 active。最可能的來源是 `multi-model-review.md` 的「收完結果就 `hapi archive-peer`」（已改寫）。另外查到兩種可能讓舊指令繼續存在的來源：

1. **HAPI 自己管理的舊 skill 副本**（目錄帶 `.hapi-managed`）。這是維護版 fork 的功能（`fork/session-control-skill`，上游沒有）：`ensureHapiSessionControlSkill` 在 `bootstrapSession` 時只把 skill 寫進**這次啟動的 flavor** 的技能目錄，失敗只記 debug log；很久沒開的 flavor 副本就一直是舊版，舊的 `hapi-session-runtime` 也沒人清。
2. **記憶裡還在教舊指令的筆記。** 同日已改寫成上表的做法；歷史紀錄保留，但標明只適用 0.30.x。

# 2026-11-18 起改由 Skillshare 發布

使用者決定 skill 一律走 Skillshare，所以：

- `hapi-session-control` 移進 shared-skills（swear01/shared-skills#66，含 `scripts/archive-peer.py` 與測試），transfer_MAC#93 bump submodule 並把 migration test 改成「必須由 Skillshare 發布」。
- fork 移除安裝器（swear01/hapi#34 合進 `main`，`9ac535a`）。`skills.ts` 的各 flavor skill roots 與測試的 HOME 重導保留。**已發布的 `v0.35.0.1` 仍含安裝器**，要等下一次 fork release 部署才會停止寫入；在那之前，Grok／Pi／agy 等沒有 Skillshare target 的目錄，開該 flavor 的 session 時會被寫回 0.35 bundled 版（內容沒有舊指令，但沒有 archive script）。Claude／Codex／Cursor／`~/.agents` 已是 Skillshare symlink，安裝器碰到 symlink 只記 debug、不覆寫。
- 遷移：把技能目錄裡帶 `.hapi-managed` 的 `hapi-session-control`／`hapi-session-runtime` 移出，再 `skillshare pull`。2026-11-18 已完成 Mac、mazu／athena、cthulhu／valkyrie、zeus、oracle；**swop 離線未做**（還要移除它 `codex`／`universal` target 對 `hapi-session-control` 的 exclude）。
- Grok、Pi、agy 都讀 `~/.agents/skills`（Pi `@earendil-works/pi-coding-agent` 與 agy binary 都有這個路徑；Grok 另讀 `~/.claude/skills`），所以不需要額外 target。

# Agent 自己 archive（archive-peer.py，2026-11-18 實測）

- 認證用環境變數 `CLI_API_TOKEN`（runner 從 `runner.env` 繼承給子程序）；Mac 的 `~/.hapi/settings.json` `cliApiToken` 對公開 hub 回 `Invalid access token`，只當備援。
- 公開 hub 在 Cloudflare 後面：Python 預設 User-Agent 會被擋（403 `error code: 1010`），要送自訂 UA。
- 已用一次性 Claude session 驗證：archive 後 `inspect-peer` 讀回 `lifecycle: archived`，再跑一次回 `already_inactive`，不存在的 id 回 `not_found`；測試 session 已 `DELETE /api/sessions/<id>`。
