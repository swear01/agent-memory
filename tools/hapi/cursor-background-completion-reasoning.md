---
title: HAPI shows Cursor background-completion reasoning in chat
scope: tools/hapi
status: verified
updated: 2026-11-27
tags: [hapi, cursor, acp, reasoning, notify_on_output]
source_refs:
  - https://github.com/tiann/hapi/issues/2267
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
structured `AGENT_NOTIFY_SUMMARY` with `status: done`. The #2267 report
includes that footer, and #2268's suppression tests start from a recognized
done footer. #2354 added a Cursor workspace rule that
asks the model to emit that footer when **Emit status summaries** is enabled.
Both changes shipped in HAPI 0.35.0.1. The Hub setting defaults to off and was
off when checked during this investigation. The preceding completion reply had
no footer, leaving the default configuration outside #2268's suppression path.

Web Chat settings can collapse Reasoning by default.

## Cursor ACP event origin

Independent review checked the Cursor CLI builds dated 2026-11-19 and 2026-11-22.
The backend can dispatch `backgroundTaskCompletionAction` before the original
`session/prompt` returns `end_turn`. Both ordinary and background-completion
text and thought updates use the same presenter without a source or completion
ID. A local presenter test produced identical ACP events for identical content
from the two paths. The incident's raw ACP stream was unavailable.

For reliable deduplication, the adapter needs a source signal and a stable
completion event identity. ACP `_meta` is one possible way to carry that
information if Cursor adds it. Keep first success and failure notifications,
new results for the same task, and replies to a soft-steer. Prompt state,
shell-exit timing, and text similarity alone can suppress real replies.
