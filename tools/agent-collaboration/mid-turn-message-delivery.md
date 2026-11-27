---
title: 執行中送訊息何時被吞掉：Codex 是 steer，Claude Code 與 Happy 是排到下一個 turn
scope: tools/agent-collaboration
status: active
confidence: high
updated: 2026-11-27
evidence: >-
  比對 codex 0.170.0 執行檔字串、Claude Code 執行檔字串，與 happy upstream
  4a21b91e 原始碼（claudeRemote 的 nextMessage、codexAppServerClient 的 RPC 集合）。
tags:
  - agent-protocol
  - happy
  - hapi
  - claude-code
  - codex
source_refs:
  - happy-up:packages/happy-cli/src/claude/claudeRemote.ts
  - happy-up:packages/happy-cli/src/utils/MessageQueue2.ts
  - happy-up:packages/happy-cli/src/codex/codexAppServerClient.ts
  - happy-up:packages/happy-wire/src/messageMeta.ts
  - happy-up:docs/plans/session-protocol-unification-v2-draft.md
  - hapi:cli/src/codex/codexRemoteLauncher.ts
  - hapi:cli/src/agent/backends/acp/AcpSdkBackend.ts
redaction: passed
---

# 契約

「送訊息會不會被 block」要拆成兩件事，混在一起就會得到錯的結論：

- sender 端的 RPC/HTTP 呼叫會不會卡住 — 三套都不卡。
- 訊息什麼時候進到模型 — 差別全在這裡。

# 三家實測對照（2026-11-27）

- **Codex 0.170.0**：app-server 有 `turn/steer`（執行檔字串 `turn/start` 21 次、`turn/steer` 6 次）。模型 system prompt 內建「The user may send a new message while you are still working. By default, treat it as steering the active task」。真正 mid-turn 注入，agent 不用等 turn 結束。
- **Claude Code**：送出進 command queue，字串自述「User-initiated sends still waiting in the command queue when this result was produced… Queued sends may coalesce into fewer turns」。sender 立刻返回，但**不**注入當前 turn，下一個 turn 才被讀。所以 Claude Code 不是 steer，是 queue。
- **Happy upstream `4a21b91e`（2026-11-26）**：`claude/claudeRemote.ts:365` 只有在收到 `result` 後才 `opts.nextMessage()`；`MessageQueue2` 是 pull 式，另有 `pushIsolateAndClear` 只給 `/clear`、`/compact`；`codex/codexAppServerClient.ts` 只有 `turn/start`、`turn/interrupt`，全 repo 沒有 `turn/steer`。`runClaude.ts` 的 `onUserMessage` handler 把訊息 push 進 queue 後就 return。結論：執行中的訊息一定等當前 turn 結束才進模型，但 sender 不會被卡。

# Happy 的 steer 只到 schema

- `packages/happy-wire/src/messageMeta.ts` 有 `expectsAcceptance`、`queuedWhileBusy`，`docs/plans/session-protocol-unification-v2-draft.md:47` 有 `steer` 欄位（並寫明「Default user messages queue for next turn」）。
- 但 `expectsAcceptance`／`queuedWhileBusy` 全 repo 只出現在 schema 與 test，沒有消費端；`steer` 只存在於草案文件。所以上游目前是「有欄位、沒行為」。

# HAPI 已有的對照實作

- `cli/src/codex/codexRemoteLauncher.ts:2094` `trySteerActiveTurn`：呼叫 app-server `steerTurn`，帶 `expectedTurnId` 消滅 stale turn，帶 `clientUserMessageId` 讓傳輸失敗後能對帳。
- `cli/src/agent/backends/acp/AcpSdkBackend.ts:743` `beginSoftSteerPrompt`：回 pending promise，不讓 hub RPC 等 turn 結束。
- `RPC_METHODS.SteerQueuedMessage`：web 上每則排隊訊息可單獨 steer。

# 若要改 Happy 的預設行為

1. 先把「可見時間」和「回執」寫進 wire：`steer` 加上 `localId`，送出的回執只代表 accepted，不代表 agent 看到。
2. 後端能力各自表態：Codex 走 `turn/steer`；Claude 走 queue，但 UI 必須顯示「排隊中」，不要假裝已注入。
3. 別把 queue 的 pull 點留在 `result` 之後才檢查 modeHash／isolate，否則訊息會被無限期 hold。
4. 測試要看得到的行為：turn 執行中送出 → ack 立即返回、且（Codex）以 `expectedTurnId` 打到 `turn/steer`。

# 陷阱

不要用「sender 沒卡」推論「agent 已經看到」。也不要假設 Claude Code 跟 Codex 一樣是 steer；字串證據顯示它是 queue。
