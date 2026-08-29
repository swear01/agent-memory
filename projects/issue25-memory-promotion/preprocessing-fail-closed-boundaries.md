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

## V2 correction and v3 gate

`source-process-shard-v2` removed image fields plus standalone and nested-JSON image data URIs and enforced the 2,700-character ceiling before tokenization. The formerly crashing source rebuilt in 14m56s, and full reconstruction verified all 44,832 occurrences. A deeper audit nevertheless found 210 empty `input_image` metadata containers. The image bytes were absent, but those markers violate a strict text-only output contract. The v2 full rebuild was stopped and is not eligible for assembly or embedding.

`source-process-shard-v3` also removes image-container metadata and payload fields such as `source`/`data`/`url`, and drops image-only containers while preserving captions and input-text siblings. Its validator explicitly requires zero residual image containers, image MIME fields, and image data URIs. Real-Qwen builder/core tests pass 13/13; the production-scale v3 pilot must pass before another global rebuild. Deep-audit artifact SHA-256: `df01cca7d76c77e8f21ffcd8780da838c0633b57a04090f9b57b4c9eae236833`.

The private harvested episodes remain immutable; only the downstream text projection must be rebuilt. Never assemble or embed the v1 or v2 shard generations.

## Metric boundary

`issue25_occurrences` counts contextual split-chunk occurrences. It is not the fixed source episode-record reference count and may exceed 10,785,150. Compute reference progress from `SUM(issue25_sources.reference_count)`.

Audit artifact on Athena: `/var/tmp/issue25-qmd-athena/shard-run/issue25-shard01-root-cause-report.json`, SHA-256 `4122da36b276ca148c801296a62c4d996268e1750595a08e8bc7087a3b2f4164`.
