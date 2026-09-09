---
title: HAPI async message delivery needs explicit settlement and stale-state guards
scope: tools/hapi
tool: HAPI web/CLI/hub
status: active
confidence: high
evidence: >-
  HAPI web and hub tests cover voice/draft reconciliation, queued-send
  settlements, cancellation races, and restart-safe ambiguous delivery.
created: 2026-08-20
updated: 2026-09-09
tags:
  - hapi
  - message-delivery
  - voice
  - concurrency
source_refs:
  - hapi:web/src/hooks/useDictation.test.ts
  - hapi:web/src/hooks/useRealtimeDictation.test.ts
  - hapi:web/src/components/AssistantChat/HappyComposer.sendError.test.tsx
  - hapi:web/src/hooks/mutations/useSendMessage.test.tsx
  - hapi:hub/src/sync/messageService.test.ts
  - hapi:hub/src/store/messages.test.ts
  - hapi:shared/src/apiTypes.ts
  - hapi:cli/src/utils/MessageQueue2.test.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Contract

Async sends must distinguish a successful commit, a cancellation, and an
ambiguous delivery state. The client must not replay a message or restore a
stale draft merely because an acknowledgement was delayed. The server-side
queue and settlement event are authoritative.

Voice/dictation sends need to reconcile the live composer text with the draft
that was captured when the send began. A failed or delayed send must preserve
both pieces of user text when they differ, then consume the final settlement
only after draft reconciliation. Clearing stale drafts is safe only after the
corresponding send is known to be settled.

Soft-steer and cancel races need epoch/turn guards and explicit reservation
ownership. Consume a reservation at dispatch, restore it only on a confirmed
failure, and make restart/reconnect outcomes idempotent. A late acknowledgement
must not overwrite newer turn state.

Scheduled sends have an admission boundary: they require a local message ID
for acknowledgement and must reject attachments before queueing. Cancellation
is idempotent; a late acknowledgement or failed steer must not restore a
message that was already cancelled.

Treat these as one delivery contract: voice draft recovery, queued-message
settlement, and soft-steer restart handling are different surfaces of the same
stale-state problem.

## Recheck before parking

A historical steer-RPC race lost a wakeup while trySteerActiveTurn awaited the app-server reply, before a waiter was installed. Before parking, recheck the steering mode and whether the queue is nonempty; re-loop only when no pending fallback owns the deferred message. Preserve the non-steerable or ended-turn fallback so it handles the message as a fresh turn.

## Verification of parking fix

The retained source reports this repair and includes raw output showing 57 launcher tests passed. This is historical source evidence, not a new execution of the HAPI regression suite.

## Spawn retry identity after ambiguous HTTP responses

`spawnPeer` 使用 `validateStatus: () => true`，所以 gateway 502/504 會走一般 response 分支，而非 transport catch；此時 Hub 可能已經建立 child。只有明確的 Hub 結構化失敗（`type: error` 且有字串 `code`）能排除一般 ambiguous response；其他錯誤、格式不完整的 success、回傳 remit ID 不符都需保留原請求的 `remitId`。清理未確認時也保留原 ID，不能信任錯誤回應中的另一個 ID。JSON、一般 CLI 文字與 MCP 錯誤皆需提供同一 ID，讓呼叫者用相同參數重試。

PR #1771 新增回歸先紅後綠：模擬 HTTP 回應遺失／異常後，以錯誤帶回的 ID 重送，檢查 request body 相同且模擬 Hub 僅有一個 spawn key；另驗證結構化失敗與兩種 CLI 輸出。這是 HTTP mock 與 CLI 單元驗證，不是實際 gateway 整合測試。
