---
title: Parallel HAPI test runs need a private TMPDIR
scope: tools/hapi/parallel-test-tmpdir
project: hapi
status: active
updated: 2026-10-21
tags: [hapi, testing, vitest, worktree]
redaction: passed
---

Concurrent `bun run test` in multiple HAPI worktrees share `/tmp/hapi-test-config.json`; ~240 CLI suites then fail with "Missing isolated hub config". Use `TMPDIR=/tmp/hapi-<task>-tmp bun run test` per worktree. Treat that failure shape as a collision, not a regression. Tell subagents up front when fanning out.
