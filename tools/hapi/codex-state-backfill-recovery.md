---
title: Recover HAPI Codex sessions from an interrupted state DB backfill
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
confidence: high
tags:
  - hapi
  - codex
  - sqlite
  - backfill
  - recovery
source_refs:
  - openai/codex:rust-v0.149.0
  - cthulhu:2026-08-26
created: 2026-08-26
updated: 2026-08-26
generated_by: openai-codex/gpt-5.6-sol
---

# Interrupted Codex state DB backfill

Codex 0.149.0 records rollout-metadata migration state in the host-local
`sqlite_home` database. A worker owns a 900-second lease. If it exits while the
row is `running`, another startup waits up to 30 seconds and may make a HAPI
app-server launch look like a runtime crash.

Confirm the runner, Codex login, `sqlite_home`, DB integrity, active Codex
processes, and the `backfill_state` row before changing anything. Do not delete
or replace a healthy database. If there is no live worker and `updated_at` is
older than the lease, start Codex directly on that host and let it run until
the state becomes `complete`; a short HAPI launch timeout is not long enough
for a large shared rollout directory.

On cthulhu, a direct Codex 0.149.0 YOLO exec safely reclaimed the expired lease,
expanded the state DB from 82 to 953 thread rows, marked the backfill complete,
and returned `OK`. `codex doctor --json` separately confirmed the installation,
auth, network, database integrity, and host-local `sqlite_home` were healthy.
