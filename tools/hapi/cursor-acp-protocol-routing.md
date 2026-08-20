---
title: Cursor ACP and legacy session protocols must remain explicitly separated
scope: tools/hapi
tool: HAPI Cursor launcher
status: active
confidence: high
evidence: >-
  Current Cursor launcher and hub metadata tests bind cursor session IDs to
  their protocol and reject unsupported legacy/ACP resume combinations.
created: 2026-08-20
updated: 2026-08-20
tags:
  - hapi
  - cursor
  - acp
  - routing
source_refs:
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.test.ts
  - hapi:hub/src/store/sessions.ts
  - hapi:hub/src/store/sessions.test.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Routing contract

New Cursor sessions use ACP. Legacy `stream-json` sessions are not silently
loaded through ACP: an unsupported resume path fails explicitly instead of
switching protocols behind the user's back.

`cursorSessionId` and `cursorSessionProtocol` are a pair. A metadata update
that writes a new session ID must drop the old protocol; an update that omits
the pair may carry both forward so archive/resume state remains usable. A
fresh ACP session must flush the metadata that pins its ACP ID before entering
the first turn, so a hub restart cannot strand the durable session handle.

Model/configuration synchronization and permission handling may be layered on
top of ACP, but must not change the protocol-selection rule or introduce a
legacy fallback after an ACP load failure.
