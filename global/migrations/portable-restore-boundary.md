---
title: Mac migration separates portable artifacts, auth state, and runtime verification
scope: global
status: active
confidence: high
evidence: Direct security instructions, selective preference checks, and explicit separation of artifact creation from unexecuted network installs.
created: 2026-08-18
updated: 2026-08-18
tags:
  - migrations
  - macos
  - secrets
  - restore
  - verification
source_refs:
  - pilot-cursor:cursor.jsonl:5
  - pilot-cursor:cursor.jsonl:9
  - pilot-cursor:cursor.jsonl:14-21
  - pilot-codex:events [1], [152]-[175]
  - pilot-hapi-export:seq=57
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Boundary

Portable backups may contain scripts, templates, and allowlisted non-secret preferences. Do not copy SSH private keys, API keys, OAuth caches, browser cookies, Keychain data, or login state.

If a secret existed in a plaintext backup, treat it as exposed: store live values only in a permission-restricted file such as `~/.secrets` with mode `0600`, and rotate or revoke the old credential. File permissions do not replace rotation.

# Restore method

Preflight each destination. Back up an existing file, directory, or symlink before replacing it, then copy only the allowlisted small preference data. Do not restore an entire application profile merely because a backup exists.

The pilot verified missing preference files before selectively copying them and confirmed their presence afterward; this does not validate a full application-state restore.

# Verification boundary

Track these states separately:

- `artifact_ready`: files or templates exist.
- `syntax_checked`: scripts parse or tests pass.
- `runtime_verified`: the installed application or service was exercised successfully.

Do not report network installation as successful from artifact existence alone, and preserve non-zero installer failures.

# Sequencing

Keep Homebrew in a single-flight installation wave. Parallelize only independent, non-Homebrew post-install tasks after that wave completes.
