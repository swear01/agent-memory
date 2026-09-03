---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-09-03
tags:
  - hapi
  - fork
  - upstream
  - release
  - permissions
source_refs:
  - public-release:hapi-fork-maintenance
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
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

Keep tokens, signing material, runner environments, and deployment configuration out of the patch series and upstream.

# Latest verified release

`v0.29.0.4` was rebuilt from upstream `980a921ba15665c54998a6ddb658103d467ff4cb` after auditing 153 open PRs (`53 carry / 96 defer / 4 drop`). The exact release commit is `dbd150922d491b3e126725c1eb1a8b5c26714d82` with source tree `6c14779972d2f5a2da7708957b2df468bf27bdf7`; PR #15, exact-main CI, release workflow `33751306214`, all eight payload digests, macOS signing, and the eight-Runner binary/connectivity rollout passed. Hub schema V29 passed `quick_check` and backup `integrity_check` before rollout. The Windows child-agent functional gate was initially omitted and must be tracked separately as described below.

Binary replacement can briefly remove `runner.state.json` while a supervised Runner completes its own handoff. Treat a missing state file during this window as transient: re-read supervisor and state together before taking action. If ownership remains split, stop the supervisor, terminate only the exact state PID after verifying it is `hapi runner start-sync` without `--started-by runner`, then start the supervisor and require its PID to match the new state PID.

# Windows Runner self-upgrade

The Windows `swop` Runner has no direct SSH management path. A standalone compiled Hub with `HAPI_UPGRADE_CHANNEL=off` also cannot materialize a `hub-artifact` by itself because it has no monorepo root. The verified narrow upgrade path is:

1. In an isolated checkout at the exact release commit, use the release-compatible Bun version, frozen dependencies, generated Web assets, and `ensureCliArtifact` to prebuild the `win32-x64` binary in the Hub artifact cache.
2. Verify the artifact SHA-256 and full source fingerprint, then temporarily run the Hub from that exact source under its existing supervisor with `HAPI_UPGRADE_CHANNEL=hub-artifact`; keep fleet policy at `alert` so no other Runner auto-upgrades.
3. Check `GET /api/upgrade/offer` through `.offer.channel`, `.offer.targetVersion`, and `.offer.targetGeneration`, then manually call `POST /api/machines/:id/upgrade-runner` only for `swop`.
4. Require a new Runner PID, the exact target version and generation, the new required capabilities, and identical active-session IDs before and after handoff. Then require live Agent availability and, for each intended installed Agent, a model-list or disposable-session canary; Runner health alone does not prove child-Agent startup. Session summaries associate a machine through `.metadata.machineId`, not a top-level `machineId`.
5. Remove the temporary override, restore the standalone Hub, and recheck the public Web, all Runner versions, active sessions, DB `quick_check`, and schema version before cleaning the isolated checkout.

For `v0.29.0.4`, the Windows artifact SHA-256 is `d73f43498b564ecaf984f6ab6ab9bf3739ac35b5b2b8e0ed487f15efef0493de` and the source generation is `102c471d67d16c3fdd037a5187ee6b1d45b5c63666f2659eeadb4dc6b41ac40c`. The `swop` handoff replaced PID `29320` with `17608`, kept the active-session set unchanged at zero, and left the restored standalone Hub healthy with upgrade channel off.

A post-rollout functional audit found that this Runner was still online while Codex availability was `invalid_configuration`; all other probed Agents were `not_found`, and the machine had zero active sessions. The HAPI update did not create the Codex override: production source only reads `HAPI_CODEX_APP_SERVER_BIN`, while both Runner handoff paths clone `process.env` into the replacement. The Agent-availability implementation is byte-identical in official upstream `980a921ba`, `v0.29.0.3`, and `v0.29.0.4` (blob `da8ed9b02dc75c7e90d4f88c1ca6cdf1f2bf6d8c`), and official `v0.29.0` already honored the same override in the app-server launcher. The same `Codex app-server exited (code=1, signal=null)` boundary failure was observed on `swop` while it still ran official `0.29.0`, before either maintained upgrade. Therefore the release artifact and self-upgrade are not the origin of the bad Codex path, but self-upgrade preserves the stale launch environment and the original acceptance check failed to detect the existing child-Agent problem. The override's exact value and persistence source remain unverified until inspected on Windows; likely sources include the Runner launch script, scheduled task, or user environment.

# v0.29.0.4 release

The published release keeps upstream at `980a921ba`, audits 153 open PRs (`53 carry / 96 defer / 4 drop`), and keeps schema V29. Seven new exact-head PRs are included: #1748, #1750, #1754, #1755, #1757, #1760, and #1761; carried #1436 and #1543 were refreshed. #1745 remains deferred because its V25-to-V26 index migration conflicts with maintained schema V29 and does not justify a standalone V30 bump.

The published source tree is `6c14779972d2f5a2da7708957b2df468bf27bdf7`. Bun 1.4.0 builds produced all five CLI targets, and the released darwin-arm64 binary reports HAPI 0.29.0.4. When using `hapi job run`, invoking a Bun 1.4.0 binary is not sufficient if package scripts call `bun` again: prepend the 1.4.0 directory to the job environment's `PATH` and avoid a login shell that rewrites it; verify the compiler path/runtime in build output.

Linux CI confirmed the release candidate, GitHub review found no major issues, and the tag-triggered release workflow produced the signed macOS CLI and desktop bundles. The macOS-local clipboard and serial Runner timing failures also reproduced on the previous exact release and remain classified as baseline test instability, not release regressions.

# Contribution boundary

Searching issues, reproducing problems, and creating an issue are separate from implementing a fix and publishing a PR. Approval to implement a fix does not imply approval to publish it.
