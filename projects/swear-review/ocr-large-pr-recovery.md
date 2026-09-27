---
title: Swear Review OCR context exhaustion on large CPAchecker PRs
scope: projects/swear-review
project: swear-review
status: active
created: 2026-09-27
updated: 2026-09-27
tags: [swear-review, ocr, cpachecker, code-search, deployment]
source_refs:
  - https://github.com/swear01/swear-review/pull/9
  - https://github.com/swear01/cpachecker/pull/288
redaction: passed
---

# Verified recovery (2026-09-27)

CPAchecker PR #288 originally failed because OCR v1.9.0 exhausted its tool-request rounds on one Java file and timed out on another. The deployed LLM gateway was current and a direct inference succeeded, so gateway reachability or version was not the cause. Upgrading OCR to v1.12.9 alone still failed: broad `code_search` results filled the context and triggered compression exhaustion.

The production fix in Swear Review PR #9 pinned OCR v1.12.9, raised the default timeout to 15 minutes, and added an optional absolute `ocr.tools_file` setting. The bundled OCR tool set omits `code_search`. PR #9 was merged as `cddb55515b88f0177ced7c4c8612c87124bb7361` and deployed on Oracle. The service health check passed after restart, with no queued or running review jobs interrupted. GitHub CI and the exact-head Gemini review passed.

The new full review of CPAchecker PR #288 at head `9442e6f97d4a8de41a68bbb9bbca856c80932857` completed both selected files with zero failed items; its Swear Review check became green. It left two code findings about repeated CFA scans and the narrow nondeterminism-function whitelist. The review gate was off, so a green infrastructure check is not a claim that those findings were fixed.

# Reuse

For a similar failure, inspect the per-item OCR manifest, actual deployed OCR binary, configured tools, and exact PR head before attributing it to the gateway. An isolated OCR trial is not production evidence. Before restarting Swear Review, check the review queue because worker shutdown aborts active jobs; then verify the deployed commit, service health, and a fresh exact-head GitHub check.
