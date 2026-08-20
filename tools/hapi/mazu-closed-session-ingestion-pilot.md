---
title: Mazu closed-session ingestion pilot boundary and outcome
scope: tools/hapi
project: hapi
tool: hapi
status: active
confidence: high
evidence: Namespace-scoped API review, paginated closed-session pilot, strict redaction audit, provenance audit, and topic extraction.
created: 2026-08-20
updated: 2026-08-20
tags:
  - hapi
  - mazu
  - sessions
  - ingestion
  - redaction
  - provenance
source_refs:
  - tools/hapi/mazu-readonly-data-access.md
  - global/workflows/memory-ingestion-ledger.md
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Durable boundary

Use Mazu's authenticated, namespace-scoped API for closed-session ingestion.
Select archived or otherwise inactive sessions only; do not bulk-read active or
thinking sessions. Fetch messages with stable `(beforeAt, beforeSeq)` cursors,
verify contiguous coverage and page checksums, then redact each complete record
before analysis.

A redacted shard is not canonical memory. Keep only a verified durable lesson in
Markdown. Credentials, authentication state, private browser/context data,
runtime metadata, nested payloads, raw API responses, and redaction markers are
never passed to topic extraction or committed.

# Pilot outcome

The pilot reviewed three archived sessions and 110 message records. Strict
record-aware redaction retained 108 records and skipped 2. Independent security
and provenance gates passed, including complete source-index mapping for retained
and skipped records.

Topic extraction found no verified transcript-derived durable lesson. Therefore
no transcript content was committed. This note records the durable ingestion
boundary and the verified negative result; it does not replace a lesson that the
source did not support.

# Reuse rule

Repeat the same closed-only, namespace-scoped, paginated, redaction-first
procedure for later Mazu batches. A later batch may add an `active` note only
when its evidence independently passes security, provenance, and duplicate/
overlap review.
