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
updated: 2026-08-20
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
