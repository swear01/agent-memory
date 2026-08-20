---
title: Shared memory ingestion ledger
scope: global
status: active
confidence: high
evidence: Read-only source inventory, context-preserving redaction, duplicate collapse, and per-machine merge review completed.
created: 2026-08-18
updated: 2026-08-20
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
| Mazu closed-session pilot | reviewed, no durable lesson | Three archived sessions were fetched through the namespace-scoped paginated API. The first redaction pass was blocked by auth/runtime markers; strict redaction retained 108 of 110 records, skipped 2, and passed independent security/provenance review. Topic extraction found no verified durable lesson; no transcript content was committed. |
| Mazu bounded closed-session recheck | reviewed, no new lesson | One closed Codex, Claude, and Cursor session was rechecked: 2,239 records, 2,234 retained, 5 skipped. Coverage, security, and provenance passed; the result matched the existing skillshare note. A larger selection was deferred after read-only API/tunnel instability, with no raw payload retained. |
| Seven-machine redacted worker pass | candidates held | Six machine-scoped workers reviewed redacted scalar-only shards. Mac and Oracle returned candidate topics for parent verification; Cthulhu, Valkyrie, Zeus, and Athena produced no safely verifiable canonical lesson. NFS duplicate views and transient metadata were excluded. See issue #9 for per-machine decisions. |
| Mazu deferred-tool smoke batch | reviewed, no durable lesson | One closed session each from Pi, OpenCode, Antigravity, and Copilot: 759 records, 757 retained, 2 skipped. Security passed; no verified new lesson was found and raw/intermediate staging was removed. See issue #10. |
| Context-preserving fleet first pass | merged selectively, coverage incomplete | The first record-level sanitizer over-filtered content: Oracle retained only 8,142 of 42,307 records. Mac merged two Unity contract updates in PR #17; Oracle merged three delivery/project contracts in PR #18. Those notes remain valid, but the first-pass no-merge conclusions are not corpus-completeness conclusions. |
| Context-preserving fleet v2.2 | reprocessed, reviewed, selectively canonicalized | A field/span-preserving sanitizer retained safe sibling context, HAPI messages, nested lesson text, and non-secret Git/SHA provenance while removing dangerous spans. Across 4,384 source files it saw 366,256 records and wrote 366,011 (244 skipped, 1 empty; 99.93% retained). The merged inline-span redactor was then applied to all seven final corpora without changing coverage: Mac changed 20 records, Zeus 1, Oracle 7, and the other four 0. All hardened JSONL and credential-shaped scans passed. Fresh machine reviews completed; no new Mac, Zeus, Valkyrie, Cthulhu, or Athena content notes were justified. A cross-project evidence-boundary note was added; HAPI updates remain separately scoped. NFS duplicate views remain one representative. |
| Additional transcript batch | blocked | Named-token or private-context text could not be proven fully removed, so no transcript content was imported. |

# Source handling policy

Raw logs are never committed or indexed. Redaction happens before subagent analysis. Workers may receive context-preserving redacted text after deterministic removal of secrets, PII, private browser/context, runtime identifiers, URLs, and local paths; raw sensitive payloads never reach workers. A source is retained only after security scanning, provenance review, exact/semantic duplicate collapse, and overlap merging.

`active` is reserved for verified durable knowledge. Unverified runtime claims remain `needs-review`; unsupported, transient, or private material remains skipped or blocked.

# QMD snapshot policy

QMD SQLite/vector indexes and model caches are disposable local state. A quiescent GPU-generated snapshot may accelerate compatible CPU destinations, but GitHub synchronizes only the canonical Markdown corpus. Never commit `index.sqlite`, WAL/SHM sidecars, models, or machine-local cache state.

# Next batches

Continue with closed sessions and public documentation in bounded, redacted, overlapping chunks. Open one machine issue per source scope, collapse exact and unnecessary semantic duplicates to one representative, and use a separate canonical PR for each accepted machine result. Record complete source coverage and merge overlapping evidence before writing notes. Active sessions and authenticated/private sources require separate review and must not be bulk-imported.
