---
title: New HAPI sessions default to YOLO permission mode
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
created: 2026-08-26
updated: 2026-10-22
generated_by: openai-codex/gpt-5.6-sol
---

# Session permission preference

When creating a new HAPI coding-agent session for this operator, use YOLO mode
unless the operator explicitly selects another permission mode. YOLO changes
approval handling only; it does not expand the requested task scope or
authorize destructive actions.

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

上游 main 的 `hapi` CLI 不能改執行中 session 的模式，也不能代按 pending
approval。開錯模式的唯一修法是 archive 後用同樣的 remit 重開。PR
`tiann/hapi#2095`（`hapi set-mode`）尚未合併。
