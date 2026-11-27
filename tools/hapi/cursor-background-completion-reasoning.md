---
title: HAPI shows Cursor background-completion reasoning in chat
scope: tools/hapi
status: verified
updated: 2026-11-27
tags: [hapi, cursor, acp, reasoning, notify_on_output]
source_refs:
  - https://github.com/tiann/hapi/pull/2268
  - https://github.com/tiann/hapi/pull/2354
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:cli/src/cursor/postCloseHarnessSilence.ts
  - hapi:hub/src/config/sessionSummaryContract.ts
  - hapi:web/src/chat/normalizeAgent.ts
---

For an English “Briefly inform” passage after a background shell completes, compare the
original message's Hub envelope with the session's metadata. The observed HAPI 0.35.0.1
session declared `metadata.flavor: cursor` and stored the passage as agent reasoning
inside a `content.type: codex` envelope. The envelope is shared by ACP output and
cannot identify the agent. The global content search did not find that reasoning
row. Searching for nearby visible task text and reading its session's message
context exposed the original row.

Cursor can run an extra assistant pass when a background shell finishes. HAPI
#2268 suppresses this speech and reasoning after the launcher recognizes a
structured `AGENT_NOTIFY_SUMMARY` with `status: done`. Its suppression tests
start from a recognized done footer. #2354 added a Cursor workspace rule that
asks the model to emit that footer when **Emit status summaries** is enabled.
Both changes shipped in HAPI 0.35.0.1. The Hub setting defaults to off and was
off when checked during this investigation. The preceding completion reply had
no footer, leaving the default configuration outside #2268's suppression path.

Web Chat settings can collapse Reasoning by default. A launcher fix would need
to preserve the first completion notification when a user is still waiting for
the background task.
