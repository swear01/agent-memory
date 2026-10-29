---
title: Parallel HAPI test runs need a private TMPDIR
scope: tools/hapi/parallel-test-tmpdir
project: hapi
status: active
updated: 2026-10-29
tags: [hapi, testing, vitest, worktree]
redaction: passed
---

Concurrent `bun run test` in multiple HAPI worktrees share `/tmp/hapi-test-config.json`; ~240 CLI suites then fail with "Missing isolated hub config". Use `TMPDIR=/tmp/hapi-<task>-tmp bun run test` per worktree. Treat that failure shape as a collision, not a regression. Tell subagents up front when fanning out. The collision also occurs between a focused suite and a full suite in the same worktree: give each concurrent test invocation its own TMPDIR, not merely each worktree. Reconfirmed 2026-10-29 while testing Antigravity model/effort changes (278 suites lost setup after another invocation removed the shared file).


Full web Vitest (306 files) can take 30+ min on the 2-core oracle host when it is busy with parallel sessions (load ~13), which is past the 30-min background-task cap. Run it with `timeout 3000 bunx vitest run --reporter=dot > log` in a background Bash with a 60-min timeout, and run focused dirs (e.g. `src/components/AssistantChat`) for quick iterations. Seen 2026-10-22 (#2085).
