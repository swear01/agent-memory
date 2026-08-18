---
title: Brave profile restoration must be process-gated and allowlisted
scope: tools/brave
tool: Brave Browser
status: draft
confidence: medium
evidence: Restore script was inspected and shell-syntax checked; live profile restoration was not exercised in the pilot.
created: 2026-08-18
updated: 2026-08-18
tags:
  - brave
  - browser
  - restore
  - migration
source_refs:
  - pilot-cursor:cursor.jsonl:1
  - pilot-cursor:cursor.jsonl:8
  - pilot-cursor:cursor.jsonl:11
  - pilot-cursor:cursor.jsonl:17
  - pilot-cursor:cursor.jsonl:21
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Process gate

Check that Brave is fully stopped before touching its profile. If a Brave process is present, skip the restore, report the skip, and rerun after quitting Brave.

# Allowlist

Restore only explicitly required non-auth profile files and allowlisted extension data. Do not copy cookies, login state, or the whole profile.

# Non-destructive replacement

Back up each existing destination path before replacing it. Restore one allowlisted subtree at a time so a failure does not destroy unrelated profile data.

# Verification

Record whether the operation was skipped, partially restored, or completed. Verify the destination after Brave starts; script creation or `bash -n` is not runtime verification.
