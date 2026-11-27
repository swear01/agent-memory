---
title: HAPI shows Cursor background-completion reasoning in chat
scope: tools/hapi
status: verified
updated: 2026-11-27
tags: [hapi, cursor, acp, reasoning, notify_on_output]
source_refs:
  - https://github.com/tiann/hapi/pull/2268
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:cli/src/cursor/postCloseHarnessSilence.ts
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
structured `AGENT_NOTIFY_SUMMARY` with `status: done`. In the observed timeline,
the earlier visible completion reply had no such footer, so the gate could not
close on it. The Hub's summary contract is opt-in. Web Chat settings can collapse
Reasoning by default; that affects display, while suppression belongs to the
Cursor launcher.
