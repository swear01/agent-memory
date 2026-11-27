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

## Foreground observation and the notification queue

In the incident, a foreground Read of the managed terminal transcript returned
the shell's successful terminal state and exit code. The assistant then checked
the artifact and reported completion. The visible tool sequence in that turn
used Read and shell commands. An Await-only fix would miss this path.

In the 2026-11-22 Cursor bundle, the interactive CLI calls the background work
registry's suppress method when a completed Await names a task. The ACP
interaction-update callback routes the tool call to the presenter without the
same TUI check. Shell progress and terminal completion have separate wakeup IDs.
Both inspected builds batch queued completions into a single follow-up and
acknowledge its IDs after it returns. Isolated probes of the packaged drain
with a simulated registry verified the batch and failure/requeue paths on
both builds. Queue acknowledgement does not establish Hub message persistence.

A proposed upstream fix can associate a foreground Await or a read of a
managed terminal's confirmed final state with the exact execution and terminal
event. Keep the completion pending until a user-visible report explicitly
covers it or a structured terminal result is committed to the transcript.
An arbitrary file path, terminal-like output, or an unrelated successful
assistant reply cannot establish this relationship.

## Which Cursor clients share this behavior

In CLI build 2026.11.22-e9042b2, interactive, print and ACP modes share the
background registry implementation and completion payload format. Each entry
point drains notifications separately. Interactive handles a terminal Await
in its UI, and print mode with stream-json writes a structured
`system/task_notification` before the follow-up. ACP uses its own drain and
presenter. A change confined to ACP affects clients running that Cursor ACP
version. Moving result accounting into shared Cursor code can cover all three
entry points while preserving their output contracts.

Ordinary CLI processes and individual ACP sessions construct separate
in-memory registry instances. A probe using the packaged registry confirmed
that suppressing a pending shell ID in one instance left the same ID in another
instance available. It also left a pending item with that ID in a different
conversation available in the same registry. This test concerns pending items.
On the HAPI side several agents share AcpSdkBackend, while the post-done
suppression rule is installed by the Cursor launcher.
