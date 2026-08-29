---
title: Issue 25 preprocessing fail-closed boundaries
scope: project
project: issue25-memory-promotion
tags: [redaction, qmd, tokenizer, sqlite, security]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# Issue 25 preprocessing fail-closed boundaries

## Inline image payloads

`image_url` with `data:image/{jpeg,png};base64,...` is browser/image state, not searchable text. A redactor that only matches `http(s)` URLs and named browser-context fields will silently retain it. Security scanning must explicitly reject residual `data:` image/audio/video payloads and runtime identifiers; never declare a corpus safe solely from ordinary URL/token/path patterns.

The 2026-08-30 failure audit found 5,594 affected episode files across 100 sources and all 24 preprocessing shards. The failed source alone contained 203,107,438 inline-image characters, 85.5% of its unique record text. A direct redactor probe showed `image_url` survived unchanged. This invalidated all mechanically completed shards for assembly and embedding.

## Bound before tokenization

When splitting an oversized Unicode record, constrain the candidate end by the character ceiling **before** constructing or tokenizing it. The failed splitter used the full record as binary-search `high`, joined a candidate spanning half the remaining multi-megabyte record, tokenized it, and only afterward checked the 2,700-character limit. This produced superlinear copying and sent multi-million-character base64 spans into llama.cpp.

For the failed source, the first-probe copying lower bound was 69,979,183,683 characters for at least 102,368 chunks. The process ran 22h44m, peaked at 5.9 GB, and ended in SIGSEGV. Long/repetitive tokenizer input has a known llama.cpp SIGSEGV class, but the exact instruction was not proven because no core artifact survived.

Required regression gates:

- data-URI image payload is removed or the record fails closed;
- no tokenizer call receives input above the configured character ceiling;
- a multi-megabyte repetitive fixture scales approximately linearly and preserves exact Unicode provenance;
- interruption resumes below the whole-source boundary;
- the full residual scanner covers data URIs and runtime IDs.

## Execution ownership and resumability

Do not run long builders as foreground SSH children. A v4 pilot's SSH transport exited 255 after 141 seconds while Athena retained the same boot ID; no matching Node process remained, and the partial SQLite database passed `quick_check` with the exact building contract and zero committed sources. This does not prove the remote termination cause, but the lifetime boundary is unsafe. Use enabled user services with exact-contract resume, bounded restart, and an `ExecCondition` that skips completed reports.

For resumability below a whole source, stage exact chunks in a private per-source SQLite database. Persist the immutable source hash, producer contract, chunk index, next record/code-point and overlap cursors, chunk text hash, and UInt32LE formatted token BLOB in bounded transactions. On resume, verify the complete staged prefix before continuing. Import only a complete source with a verified ordered digest into the shard DB in one source-atomic transaction; require interrupted and uninterrupted runs to have byte-identical logical digests.

## V2 correction and v4 gate

`source-process-shard-v2` removed image fields plus standalone and nested-JSON image data URIs and enforced the 2,700-character ceiling before tokenization. The formerly crashing source rebuilt in 14m56s, and full reconstruction verified all 44,832 occurrences. A deeper audit nevertheless found 210 empty `input_image` metadata containers. The image bytes were absent, but those markers violate a strict text-only output contract. The v2 full rebuild was stopped and is not eligible for assembly or embedding.

`source-process-shard-v4` covers OpenAI `input_image`/`image_url`/`b64_json`, Anthropic image `source` blocks, Gemini image MIME plus inline `data`, embedded image data URIs, image-container metadata, and image-only containers while preserving captions, revised prompts, and input-text siblings. Its validator explicitly requires zero residual image containers, image MIME fields, and image data URIs. Real-Qwen builder/core tests pass 14/14.

The production-scale v4 pilot completed the formerly failing source in 12m59s with zero service restarts: 18,438 records, 21,642 references, 44,229 occurrences, and 41,825 exact contextual/token inputs. Full reconstruction and independent deep audit passed with zero media data URIs, image containers, image MIME fields, image payload keys, hash mismatches, provenance failures, token-BLOB failures, or character/token-limit failures. Build/validation/deep-audit SHA-256 values are `df88bc64a1f4539cca9d16a77ec19b15ffe3a02730fbdc0f06f6e06786995812`, `04099ef97c1e3b66dcc6a45e62198bbd8ea01f953cbfe5d052b13a22ce5e147d`, and `a0ca82bba4be1586d36388e96dbb878b794f9127510a1733a3caba14cac010f4`. The earlier v2 deep-audit artifact remains `df01cca7d76c77e8f21ffcd8780da838c0633b57a04090f9b57b4c9eae236833`.

The private harvested episodes remain immutable; only the downstream text projection must be rebuilt. Never assemble or embed the v1 or v2 shard generations.

## Metric boundary

`issue25_occurrences` counts contextual split-chunk occurrences. It is not the fixed source episode-record reference count and may exceed 10,785,150. Compute reference progress from `SUM(issue25_sources.reference_count)`.

Audit artifact on Athena: `/var/tmp/issue25-qmd-athena/shard-run/issue25-shard01-root-cause-report.json`, SHA-256 `4122da36b276ca148c801296a62c4d996268e1750595a08e8bc7087a3b2f4164`.
