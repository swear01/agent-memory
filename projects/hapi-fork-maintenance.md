---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-09-02
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

`v0.29.0.3` was rebuilt from upstream `980a921ba` after auditing 141 open PRs (`46 carry / 91 defer / 4 drop`). The exact release commit is `a6a0c03d2`; tag tests, the release workflow, asset digests, and the seven-Runner deployment passed. Hub schema V29 passed `quick_check` and `integrity_check`, and all 19 recorded pre-deployment session roots survived.

Binary replacement can briefly remove `runner.state.json` while a supervised Runner completes its own handoff. Treat a missing state file during this window as transient: re-read supervisor and state together before taking action. If ownership remains split, stop the supervisor, terminate only the exact state PID after verifying it is `hapi runner start-sync` without `--started-by runner`, then start the supervisor and require its PID to match the new state PID.

# Windows Runner self-upgrade

The Windows `swop` Runner has no direct SSH management path. A standalone compiled Hub with `HAPI_UPGRADE_CHANNEL=off` also cannot materialize a `hub-artifact` by itself because it has no monorepo root. The verified narrow upgrade path is:

1. In an isolated checkout at the exact release commit, use the release-compatible Bun version, frozen dependencies, generated Web assets, and `ensureCliArtifact` to prebuild the `win32-x64` binary in the Hub artifact cache.
2. Verify the artifact SHA-256 and full source fingerprint, then temporarily run the Hub from that exact source under its existing supervisor with `HAPI_UPGRADE_CHANNEL=hub-artifact`; keep fleet policy at `alert` so no other Runner auto-upgrades.
3. Check `GET /api/upgrade/offer` through `.offer.channel`, `.offer.targetVersion`, and `.offer.targetGeneration`, then manually call `POST /api/machines/:id/upgrade-runner` only for `swop`.
4. Require a new Runner PID, the exact target version and generation, the new required capabilities, and identical active-session IDs before and after handoff. Session summaries associate a machine through `.metadata.machineId`, not a top-level `machineId`.
5. Remove the temporary override, restore the standalone Hub, and recheck the public Web, all Runner versions, active sessions, DB `quick_check`, and schema version before cleaning the isolated checkout.

For `v0.29.0.3`, the Windows artifact SHA-256 was `d439f3d18b004ed401d7bdd13cce499ea32bf176ca2cbfcb4bf55ad8380b90b7` and the source generation was `7170b7a8be2ca0f175810408de96cb9e23aa2e3dc74fd778f6cfef202c4a6877`. The Runner lock handoff completed and its existing active session survived unchanged.

# Contribution boundary

Searching issues, reproducing problems, and creating an issue are separate from implementing a fix and publishing a PR. Approval to implement a fix does not imply approval to publish it.
