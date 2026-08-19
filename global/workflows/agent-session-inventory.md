---
title: Seven-machine coding-agent session inventory and local error index
scope: global
status: active
confidence: high
evidence: Read-only inventory completed across the seven HAPI machines; raw sources were not modified.
created: 2026-08-20
updated: 2026-08-20
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
