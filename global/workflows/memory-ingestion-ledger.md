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

# Codex batch B001

Run: `20260818-codex-b001`

| Source | Result | Notes |
|---|---|---|
| Codex / WirePulling | extracted | `projects/WirePulling/ab-logic-presentation-contract.md` and `projects/WirePulling/unity-validation-boundary.md`; proposals and unresolved MCP runtime claims were not promoted. |
| Codex / game-chain-2026 | extracted | `projects/game-chain-2026/unity-mcp-verification-boundary.md` remains `needs-review` because Unity runtime handshake was unavailable. |
| Codex / linked_verseDX | extracted | Colima CPU triage note active; logind root trigger remains unresolved. |
| Codex / computer-vision-course | extracted | Narrated PPTX pipeline note remains `needs-review`; playback and MP4 export were not verified. |
| Codex / transfer_MAC | blocked-secret | Embedded handover contained non-placeholder credential material; no note was written and the staged copy was removed. |
| Codex / playground | blocked-private-data | Authenticated browser research contained private identities, listings, and cookie/CSRF context; no note was written and the staged copy was removed. |

# Pi batch B001

Run: `20260818-pi-b001`

Three old Pi sessions were selected for processing, but all were blocked before subagent analysis because deterministic redaction could not prove that named token/private-context text was fully removed. No Pi raw or redacted content was imported. Revisit only with a stronger sanitizer and an explicit security review.

# Snapshot pilot

A quiescent QMD 2.8.3 SQLite baseline with 13 documents and 13 vectors was created on the idle GPU host Athena and copied to the NFS fleet, Zeus, and Oracle. Each target ran local `qmd update`; all reported 13 documents and 13 vectors, and Oracle's CPU-only structured semantic query retrieved the copied memory. Target database checksums changed after local metadata/path synchronization, which is expected. No index, model, WAL, or SHM files were committed to Git.

# Next batches

1. Closed Codex sessions, grouped by project and machine.
2. Pi sessions and Cursor transcripts.
3. Claude and OpenCode stores.
4. Antigravity and ancillary harnesses.
5. HAPI exports used to link provenance, not as a bulk diagnostic-log corpus.
6. Selected GitHub gists, repository docs, and Playground/Flutter documentation.
