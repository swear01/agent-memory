---
title: Shared NFS HAPI fleets need per-machine identity and local writable state
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
evidence: Live NFS audit, five-host cutover, 68 SQLite quick checks, and documented hub outage recovery.
created: 2026-08-18
updated: 2026-08-18
tags:
  - hapi
  - nfs
  - sqlite
  - wal
  - fleet
source_refs:
  - pilot-hapi-gist:hapi-gist.md:77-104
  - pilot-hapi-gist:hapi-gist.md:165-200
  - pilot-hapi-gist:hapi-gist.md:438-444
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Machine identity

Shared NFS home may contain common configuration, but each runner needs its own runtime `HAPI_HOME` under local storage such as `/var/tmp/hapi-<host>`. Never use a shared fallback identity as a live runner identity.

Validate the per-host runtime settings, runner state, supervisor ownership, and hub machine record together.

# SQLite and WAL

SQLite databases using WAL must remain on the same host as their processes. Keep Codex, OpenCode, and Cursor writable runtime state on local storage; retain NFS data only as a seed when local state is empty. Never synchronize an active local SQLite database back to NFS.

A five-host cutover moved the writable state locally and passed `PRAGMA quick_check` on 68 databases without new damage or locking errors.

# Mazu hub

The hub must use a local database such as `/var/tmp/hapi-hub/hapi.db`, not a live NAS database. If the local copy is missing, recreate it through the managed SQLite backup path.

Recovery order is:

1. Check the hub supervisor and local HTTP.
2. Verify the hub uses its local `HAPI_HOME`.
3. Recreate the local DB through the managed setup command if needed.
4. Confirm the tunnel supervisor.
5. Verify the public endpoint last.
