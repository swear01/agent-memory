---
title: New HAPI sessions default to Auto permission mode
scope: tools/hapi
project: hapi
tool: session creation
status: active
confidence: high
tags:
  - hapi
  - sessions
  - permissions
  - user-preference
source_refs:
  - user-confirmed:2026-08-26
  - user-confirmed:2026-10-22
  - user-confirmed:2026-10-25
  - user-confirmed:2026-11-06
created: 2026-08-26
updated: 2026-11-14
generated_by: openai-codex/gpt-5.6-sol
---

# Session permission preference

When creating a new HAPI coding-agent session for this operator, use Auto mode
unless the operator explicitly selects another permission mode: Claude
`auto`, Codex `autoReview` (Codex rejects `auto`). Always pass the mode
explicitly; an omitted mode starts Claude in `default` (Manual). Symptom of a missed flag on a `spawn-peer` child (2026-11-12, valkyrie): the child stays `thinking` with zero assistant messages and no file changes for 30+ min, because its first tool call waits for an approval nobody sees; archive it and respawn with `--permission-mode auto`. Also pass `hapi wait-peer --timeout <s>` for long remits; the default timeout returns `ok:false` while the child is still working. Use YOLO only
when the operator asks for it. The permission mode changes approval handling
only; it does not expand the requested task scope or authorize destructive
actions.

2026-10-25：使用者把預設從 YOLO 改成 Auto（「Auto 是蠻好的」）。

After creation, verify the recorded session permission mode together with the
agent, model, machine, workspace, and ready state.

For fleet deployment, independently stored Skillshare sources may remain on a
clean detached merge commit after `skillshare pull`. Verify that the published
`origin/main` commit is an ancestor and that the actual target contains the
rule; exact per-host HEAD equality is not a valid gate.

## 從 Claude Code 派 Claude session 時

`hapi spawn-peer` 一定要明確帶 `--permission-mode`。沒帶的話 Claude session
會以 `default` 開，每個工具呼叫都停下來等人按 allow。2026-10-22 一次開出四個
issue session 全卡在 pending，只能 archive 後重開。

Claude flavor 的 YOLO 是 `bypassPermissions`（`shared/src/modes.ts` 的 Claude
catalog）。但呼叫端 Claude Code 在 auto mode 下，classifier 會以「Create Unsafe
Agents」擋掉 `spawn-peer --permission-mode bypassPermissions`。這種情況下改用
`--permission-mode auto`，使用者可接受。要 YOLO 就得由使用者加 permission rule，
或在 web UI 切換。

2026-11-06 已在維護版 HAPI CLI `0.30.7.7` 實測
`hapi set-mode <exact-session-id> auto` 成功，回覆 session 已切換為 auto。
因此此版本開錯模式時可先用這個命令修正，不必直接 archive。這只證明
維護版支援，不延伸宣稱上游版本已支援。

2026-11-12：`set-mode` 只影響之後的工具呼叫。在 default 模式下已經送出、正在等批准的請求不會因此通過（Hub `agentState.requests` 仍有 1 筆），session 會一直顯示 `thinking: true`，卻沒有任何進度。修正方式是 `hapi abort-peer <id>` 中斷這一輪，再用 `ping-peer` 叫它繼續；之後檢查 `requests` 應為 0。最根本的還是 spawn 時就要帶 `--permission-mode auto`。

## MCP `spawn_peer` 與「Default = Manual」（2026-10-25）

MCP 的 `spawn_peer` 工具同樣要明確帶 `permissionMode`（例如 `"auto"`）。省略時
runner 不加旗標，`hapi claude` 自己補 `'default'`，SDK wrapper 再明確傳
`--permission-mode default` 給 Claude。HAPI 走的是 SDK/stream-json 路徑，所以
Claude Code 互動模式的預設（v2.1.283 起是 Auto）不會生效，自己的
`~/.claude/settings.json` `defaultMode` 也會被這個明確參數蓋掉。2026-10-25 一個
orchestrator 用 MCP 開的 B session 因此開在 Manual。

Claude Code 現在把 `default` 叫 **Manual**；`dontAsk` 是「會詢問的一律拒絕」，
不是 YOLO，HAPI 的 Yolo 對應 `bypassPermissions` 是對的。使用者決定**不改** HAPI
省略時的預設，由 agent 自己選模式；`tiann/hapi#2166` 只把 Claude 的標籤改成
Manual。

## Claude Code 審查也用 Auto（2026-11-06）

使用者再次明確要求：「開 session 要開 auto」，並要求更新記憶與 QMD。
新建 Claude Code session，包含無人看管的審查，一律明確傳
`--permission-mode auto`，除非使用者當次指定其他模式。這取代 2026-10-29
對 Claude 審查使用 `bypassPermissions` 的舊例外；其他 agent 的模式另按其合法值。

本次教訓：漏傳旗標會以 `default`（Manual）啟動，Claude 幾秒內已產生 Bash
工具呼叫，卻停在 permission request；HAPI `inspect-peer` 只顯示 `thinking=true`，
`wait-peer` 最後 timeout。不能據此說 Claude 正在長時間檢查數學，也不能直接歸檔。
先查本次自己建立的 session 是否卡在工具核准，再依已授權的 Auto 偏好修正模式。
不要把排除 Bash、改用 Read 當作 Auto 設定的替代品。

## HAPI 0.35（2026-11-14，valkyrie 實測 `0.35.0.1`）

CLI 已移除 `spawn-peer`、`wait-peer`、`archive-peer`（`hapi archive-peer` 回 "Unknown command"）；派工改用 MCP 工具 `spawn_peer`（`waitForReply: true` 後結束這一輪，回覆會以新訊息送回），讀取用 `inspect_peer`，追問用 `ping_peer`。Archive 不要丟給使用者（使用者 2026-11-14 明確反對）：CLI 沒了但 hub API 還在。用 `$HAPI_HOME/settings.json` 的 `cliApiToken` 打 `POST $HAPI_API_URL/api/auth`（body `{"accessToken": ...}`）換 JWT，再 `POST /api/sessions/<id>/archive`（`authorization: Bearer <jwt>`），回 `{"ok":true}` 後 session 變 `active=false`。只 archive 自己開、已閒置的 worker。`spawn_peer` 的工具說明寫預設是 yolo；依本偏好仍要明確傳 `permissionMode: "auto"`（2026-11-14 有兩個 CULS worker 沒傳而以 yolo 開出）。
