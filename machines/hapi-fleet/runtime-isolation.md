---
title: Shared filesystem agent fleets need per-runner identity and local writable state
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
evidence: Repeated fleet audits, SQLite quick checks, and documented recovery rehearsals.
created: 2026-08-18
updated: 2026-08-19
tags:
  - hapi
  - nfs
  - sqlite
  - wal
  - fleet
source_refs:
  - public-release:runtime-isolation
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Runtime identity

A shared filesystem may carry common configuration, but every runner needs its own host-local runtime directory and identity. Never use a shared fallback identity as a live runner identity.

Validate the runner's local runtime settings, state ownership, supervisor, and hub registration together.

# SQLite and WAL

SQLite databases using WAL must remain on the same host as their processes. Keep writable agent runtime state on local storage; use shared storage only as a seed when local state is empty. Never synchronize an active local SQLite database back to shared storage.

Before a cutover, run `PRAGMA quick_check` on each database and verify that no process still points at the old shared writable state.

# Hub recovery

The hub should use a host-local database, not a live shared-storage database. If the local copy is missing, recreate it through the managed backup path.

Recovery order is:

1. Check the hub supervisor and local HTTP endpoint.
2. Verify the hub uses its host-local runtime directory.
3. Recreate the local database through the managed setup command if needed.
4. Confirm the tunnel supervisor.
5. Verify the public endpoint last.
