---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, force-with-lease rewrite, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-08-18
tags:
  - hapi
  - fork
  - upstream
  - release
  - permissions
source_refs:
  - pilot-hapi-gist:hapi-gist.md:22-27
  - pilot-hapi-gist:hapi-gist.md:364-372
  - pilot-hapi-gist:hapi-gist.md:529-556
  - pilot-hapi-gist:hapi-gist.md:590-626
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Rebuild policy

Rebuild the maintained branch from the latest upstream base plus an explicitly audited patch queue and the current maintenance-version patch.

Before each release, rescan current upstream PRs and record `carry`, `drop`, or `defer` decisions with the live PR head, review state, maintainer signal, scope, and behavior rationale. Do not replay an old queue blindly.

# Gates

A rehearsal proving tree equality, tests, or artifacts does not authorize publication. Require:

- isolated rehearsal
- frozen install and full tests
- artifact, checksum, and signing verification
- explicit operator approval
- `--force-with-lease` using the captured old branch SHA

Keep private tokens, signing material, runner environments, and fleet configuration out of the patch series and upstream.

# Contribution boundary

Searching issues, reproducing problems, and creating an issue are separate from implementing a fix and publishing a PR. Approval to implement a fix does not imply approval to publish it.

Fleet, Cloudflare, supervisor, host-path, and token configuration remains private distribution data.
