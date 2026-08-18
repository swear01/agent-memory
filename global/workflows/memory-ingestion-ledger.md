---
title: Shared memory ingestion ledger
scope: global
status: active
confidence: high
evidence: Read-only fleet inventory and redacted pilot run completed.
created: 2026-08-18
updated: 2026-08-18
tags:
  - shared-memory
  - ingestion
  - provenance
  - checkpoints
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Purpose

Track which agent-history and documentation sources have been examined, summarized, committed, skipped, or deferred. Raw source files and detailed hashes remain in the local run manifest under `/var/tmp`; this ledger contains only redacted source identifiers and durable status.

# Pilot run

Run: `20260818-pilot`

| Source | Result | Notes |
|---|---|---|
| Pi pilot | skipped | `/exit` only; no durable lesson. |
| Claude pilot | skipped | One OAuth-expiry failure with no resolution. |
| Codex pilot | extracted | `tools/computer-use/mac-restore.md` written; other migration candidates deferred or rejected. |
| Cursor pilot | extracted | `global/migrations/portable-restore-boundary.md` written; unverified restore candidates remain drafts or deferred. |
| HAPI export pilot | extracted | `projects/transfer_MAC/skillshare.md` written; legacy target-overlap warning remains documented. |
| HAPI documentation pilot | extracted | `machines/hapi-fleet/runtime-isolation.md`, `tools/hapi/supervisor-safe-operations.md`, and `projects/hapi-fork-maintenance.md` written. |

# Processing policy

Process closed/stable sessions first. Redact credentials, auth state, encrypted reasoning, private keys, cookies, and machine-specific secrets before any model or subagent reads the source. Treat active sessions as metadata-only until they become stable.

A source is not complete merely because it was discovered or parsed. Record extraction, review, Markdown write, QMD verification, Git commit, and Git push as separate checkpoints. Keep unverified or conflicting candidates as `needs-review` instead of presenting them as active facts.

# Snapshot pilot

A quiescent QMD 2.8.3 SQLite baseline with 13 documents and 13 vectors was created on the idle GPU host Athena and copied to the NFS fleet, Zeus, and Oracle. Each target ran local `qmd update`; all reported 13 documents and 13 vectors, and Oracle's CPU-only structured semantic query retrieved the copied memory. Target database checksums changed after local metadata/path synchronization, which is expected. No index, model, WAL, or SHM files were committed to Git.

# Next batches

1. Closed Codex sessions, grouped by project and machine.
2. Pi sessions and Cursor transcripts.
3. Claude and OpenCode stores.
4. Antigravity and ancillary harnesses.
5. HAPI exports used to link provenance, not as a bulk diagnostic-log corpus.
6. Selected GitHub gists, repository docs, and Playground/Flutter documentation.
