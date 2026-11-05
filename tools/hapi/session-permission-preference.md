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
updated: 2026-11-06
generated_by: openai-codex/gpt-5.6-sol
---

# Session permission preference

When creating a new HAPI coding-agent session for this operator, use Auto mode
unless the operator explicitly selects another permission mode: Claude
`auto`, Codex `autoReview` (Codex rejects `auto`). Always pass the mode
explicitly; an omitted mode starts Claude in `default` (Manual). Use YOLO only
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
