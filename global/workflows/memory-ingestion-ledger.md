---
title: Shared memory ingestion ledger
scope: global
status: active
confidence: high
evidence: Read-only source inventory and redacted pilot runs completed.
created: 2026-08-18
updated: 2026-08-19
tags:
  - shared-memory
  - ingestion
  - provenance
  - checkpoints
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Purpose

Track which source batches have been examined, summarized, committed, skipped, or deferred. Detailed source paths and hashes remain in local run manifests and are not part of this public repository.

# Public-safe batch summary

| Batch | Result | Public-safe note |
|---|---|---|
| Initial pilot | extracted | Durable lessons from several agent tools were written; transient and unsupported candidates were omitted. |
| Project batch B001 | extracted | Project-specific notes were written only where the evidence was durable and sufficiently verified. |
| Authenticated/private-context sources | blocked | Sources containing credentials, authenticated browser state, private identities, or private listings were not imported. |
| Runtime-only sessions | skipped | Generic startup metadata and transient authentication failures produced no durable note. |
| OpenCode configuration batch | extracted | Configuration discovery was written as active; ACP recovery remains `needs-review` because runtime recovery was not verified. |
| Additional transcript batch | blocked | Named-token or private-context text could not be proven fully removed, so no transcript content was imported. |

# Source handling policy

Raw logs are never committed or indexed. Redaction happens before subagent analysis. A source is retained only after deterministic secret scanning, private-context review, provenance review, and duplicate/overlap merging.

`active` is reserved for verified durable knowledge. Unverified runtime claims remain `needs-review`; unsupported, transient, or private material remains skipped or blocked.

# QMD snapshot policy

QMD SQLite/vector indexes and model caches are disposable local state. A quiescent GPU-generated snapshot may accelerate compatible CPU destinations, but GitHub synchronizes only the canonical Markdown corpus. Never commit `index.sqlite`, WAL/SHM sidecars, models, or machine-local cache state.

# Next batches

Continue with closed sessions and public documentation in bounded, redacted, overlapping chunks. Record complete source coverage and merge overlapping evidence before writing notes. Active sessions and authenticated/private sources require separate review and must not be bulk-imported.
