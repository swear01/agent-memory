---
title: Seven-machine coding-agent session inventory and local error index
scope: global
status: active
confidence: high
evidence: Read-only inventory completed across the seven HAPI machines; raw sources were not modified.
created: 2026-08-20
updated: 2026-09-10
tags:
  - shared-memory
  - session-search
  - error-index
  - hapi-fleet
  - working-directories
---

# Inventory boundary

The canonical `agent-memory` Markdown collection is a distilled knowledge base,
not the complete agent-history corpus. A read-only scan covered all seven HAPI
machines and the installed agent stores.

Shared filesystems require source-path deduplication. Writable Codex, OpenCode,
and Cursor runtime state must remain host-local; shared locations are seed-only
and must not be treated as live databases. Inventory counts, raw paths, and
machine-specific metadata remain in local manifests rather than this public
repository.

# Local search index

A redacted, machine-local SQLite FTS5 index may be built for session metadata,
workspace evidence, scan provenance, and bounded redacted error candidates. It
is disposable local state, not canonical memory and not a fleet synchronization
layer.

Error candidates are retrieval signals only. Repeated tool output, user prompts
that mention an error, and known-successful retries can match the same pattern.
A candidate becomes an `active` memory case only after its root cause and
resolution are independently verified.

# Runtime boundary

Do not copy raw messages, credentials, cookies, OAuth state, private browser
context, or agent databases into Git. Read-only inventory is not permission to
repair shared databases or synchronize them back to a host-local runtime.
Invalid or ambiguous database artifacts remain blocked until a separate,
quiescent, read-only investigation proves their provenance and safety.

# Provenance

Detailed scan manifests and source hashes remain outside the public repository.
Only durable, public-safe boundaries and verified lessons belong in canonical
Markdown.

# Session status census boundaries

A dispatch ledger covers the sessions recorded in that ledger. Even a fresh
Hub check of every row does not establish that all sessions for the repository
are archived: an independently created research chat can share the same working
directory or title and be absent from the dispatch ledger. Report the checked
population, observation time and exclusions explicitly.

When the user supplies an exact session reference, inspect it directly and add
it to a separate referenced-session inventory; do not retroactively label it as
a coordinator-created child. Follow the applicable session-control permission
boundaries for discovery and operations on unrelated sessions.

Keep runner-active, agent-thinking, lifecycle and owned background processes
separate. An active runner with thinking=false can be an idle finished chat.
Before archiving a completed session, preserve its handoff, check owned work and
review durable-memory needs; synchronize any memory/QMD changes before final
closeout. Archive preserves the conversation; it is not deletion of research
artifacts or authorization to remove a shared dirty checkout.

# Nested delegated sessions

Register every successful child-spawn receipt with parent, child and remit IDs,
including failed and superseded verification attempts. A top-level dispatch
ledger can omit grandchildren even when every listed parent is archived.
Closing the later successful test child does not close the earlier failed one.
Check each known descendant's live lifecycle before accepting parent cleanup.

Preserve failed-run commands and outcomes separately from the later passing
verification. A changed command and a passing result do not alone establish
why the earlier attempt failed. Before removing a shared task worktree, verify
that no parent or child session still depends on its CWD. A runner can remain
alive with a deleted CWD; do not recreate or rerun completed work merely to
archive it when root can verify terminal evidence and owned processes directly.

When reconciling omissions, a bounded scan of registered worktree rollouts for
successful spawn receipts can recover child IDs without enumerating unrelated
chats. State the dates, stores, matching rules and unsupported receipt forms;
such a scan is not automatically an exhaustive Hub census.
