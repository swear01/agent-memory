---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-09-04
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

`v0.29.0.5` was rebuilt from upstream `980a921ba15665c54998a6ddb658103d467ff4cb` after auditing 154 open PRs (`54 carry / 96 defer / 4 drop`). The exact release commit is `7a89deefb2cbca900ba54eed1f4e399fada52bb2` with source tree `85ad6564dc84c759099523e80d522fd38df2b371`; PR #16, exact-main review and CI, release workflow `33793933802`, tag test `33793933807`, all eight payload digests, and macOS signing passed. All eight active Runners were then verified live at `0.29.0.5`, including systemd, launchd, and PM2 ownership; Hub/Tunnel and local/public Web behavior; served asset version; schema V29 `quick_check`; and workspace-root guard `403`/`200` behavior with the test row removed. The Mac kept seven session roots and its pre-rollout long-running jobs through handoff. Canonical HAPI Skillshare guidance was published in `shared-skills` commit `913d618fce329ab6832abfce722b56225a3370e2` and transfer repository pointer `7abe2ff5c7ed3ffd077bf843bb144bdae86b7b15`, then sync-verified on the seven non-Windows hosts.

The `v0.29.0.5` app shell returned no `Cache-Control: no-store` header through either the Hub-local or public route during rollout verification. Treat this as an unresolved Web-cache observation, not as a failed Runner deployment or a confirmed regression, until compared with the intended header contract and a previous release.

Binary replacement can briefly remove `runner.state.json` while a supervised Runner completes its own handoff. Treat a missing state file during this window as transient: re-read supervisor and state together before taking action. If ownership remains split, stop the supervisor, terminate only the exact state PID after verifying it is `hapi runner start-sync` without `--started-by runner`, then start the supervisor and require its PID to match the new state PID.

# Windows Runner self-upgrade

The Windows `swop` Runner has no direct SSH management path. A standalone compiled Hub with `HAPI_UPGRADE_CHANNEL=off` also cannot materialize a `hub-artifact` by itself because it has no monorepo root. The verified narrow upgrade path is:

1. In an isolated checkout at the exact release commit, use the release-compatible Bun version, frozen dependencies, generated Web assets, and `ensureCliArtifact` to prebuild the `win32-x64` binary in the Hub artifact cache.
2. Verify the artifact SHA-256 and full source fingerprint, then temporarily run the Hub from that exact source under its existing supervisor with `HAPI_UPGRADE_CHANNEL=hub-artifact`; keep fleet policy at `alert` so no other Runner auto-upgrades.
3. Check `GET /api/upgrade/offer` through `.offer.channel`, `.offer.targetVersion`, and `.offer.targetGeneration`, then manually call `POST /api/machines/:id/upgrade-runner` only for `swop`.
4. Require a new Runner PID, the exact target version and generation, the new required capabilities, and identical active-session IDs before and after handoff. Then require live Agent availability and, for each intended installed Agent, a model-list or disposable-session canary; Runner health alone does not prove child-Agent startup. Session summaries associate a machine through `.metadata.machineId`, not a top-level `machineId`.
5. Remove the temporary override, restore the standalone Hub, and recheck the public Web, all Runner versions, active sessions, DB `quick_check`, and schema version before cleaning the isolated checkout.

For `v0.29.0.4`, the Windows artifact SHA-256 is `d73f43498b564ecaf984f6ab6ab9bf3739ac35b5b2b8e0ed487f15efef0493de` and the source generation is `102c471d67d16c3fdd037a5187ee6b1d45b5c63666f2659eeadb4dc6b41ac40c`. The `swop` handoff replaced PID `29320` with `17608`, kept the active-session set unchanged at zero, and left the restored standalone Hub healthy with upgrade channel off.

For `v0.29.0.5`, the Windows artifact SHA-256 is `d185e942aef97b6a5d6a2d62be26d2430d71f0882c4e2ee2842f45048684117e` and the source generation is `0b70ba38c0228b984f780485ae7418d9ea5235b99f043c485b1d4fe0e7e5f7c8`. The `swop` handoff replaced PID `10864` with `7944`, preserved its existing active session, and left the restored standalone Hub healthy with upgrade channel off and fleet policy `alert`.

A post-rollout functional audit found that this Runner was still online while Codex availability was `invalid_configuration`; all other probed Agents were `not_found`, and the machine had zero active sessions. The HAPI update did not create the Codex override: production source only reads `HAPI_CODEX_APP_SERVER_BIN`, while both Runner handoff paths clone `process.env` into the replacement. The Agent-availability implementation is byte-identical in official upstream `980a921ba`, `v0.29.0.3`, and `v0.29.0.4` (blob `da8ed9b02dc75c7e90d4f88c1ca6cdf1f2bf6d8c`), and official `v0.29.0` already honored the same override in the app-server launcher. The same `Codex app-server exited (code=1, signal=null)` boundary failure was observed on `swop` while it still ran official `0.29.0`, before either maintained upgrade. Therefore the release artifact and self-upgrade are not the origin of the bad Codex path, but self-upgrade preserves the stale launch environment and the original acceptance check failed to detect the existing child-Agent problem.

On 2026-09-04, local Windows inspection verified the persistence source: the Scheduled Task launch script at `%LOCALAPPDATA%\Programs\Hapi\start-hapi-runner.ps1` explicitly assigned `HAPI_CODEX_APP_SERVER_BIN` to a deleted, version-specific Codex Desktop app path. Process, User, and Machine environment scopes had no persistent value, while the normal `codex` on `PATH` and `codex app-server` both worked. The repair removed only that override assignment, preserved the Runner's other launch parameters and dynamic `PATH` setup, confirmed zero active sessions, and restarted through the existing supervisor. A post-restart HAPI-managed Codex canary successfully executed `Get-Location`, returned the Windows user home, and was archived; the machine then had zero active sessions. Prefer removing this override when normal `PATH` discovery works instead of repointing it to another version-specific installation path.

# Windows `swop` operations onboarding

On 2026-09-04, `swop` was brought into the shared Skillshare and QMD workflow. Its clean `shared-skills` checkout fast-forwarded six commits to `2525b8ed3ca12ea3bc332b3bdfce0a335fb3cd4f`; Skillshare `0.20.27` then reported 30 active skills, two targets, 54 junctions, and zero broken, out-of-source, or drifted links while preserving `.codex\skills\.system`. Node `24.19.0`, npm `11.17.0`, QMD `2.8.3`, and a clean HTTPS checkout of `agent-memory` were added. The `memory` collection indexed 92 Markdown files and a BM25 canary found `hapi-fork-maintenance.md`. `.agents\AGENTS.md` is now the single hardlinked rules source for Codex, Claude, OpenCode, and Pi; all five paths share SHA-256 `AA2918D991D9E68DF9EC4F38C0AC704E093E0EA50BCCC5277BE3412631E933A9`.

Do not treat Windows QMD vector embedding as healthy yet. A CPU run remained at zero vectors for about 28 minutes despite consuming CPU, matching upstream Windows hang reports #679 and #739; only that QMD child process was interrupted, leaving the lexical index usable. The host now resolves Codex `0.152.0`, Claude `2.1.258`, OpenCode `1.18.27`, AGY `1.1.25`, Grok `1.0.5`, and Pi `0.84.4`; Gemini and Cursor Agent remain absent because Gemini is retired and no confirmed official Windows Cursor Agent package was found. Preserve newer working official versions rather than rolling them back merely to match an older fleet snapshot.

At the onboarding checkpoint, the HAPI launcher and Runner both reported `0.29.0.4`; Hub state recorded active Runner PID `10864`, the expected generation `102c471d67d16c3fdd037a5187ee6b1d45b5c63666f2659eeadb4dc6b41ac40c`, supervised restart, and no last spawn error. The existing Codex session process PID `4116` survived that graceful Runner restart and remained responsive, but the replacement Runner's local session list did not re-adopt it. The later `v0.29.0.5` exact-source handoff did preserve its existing HAPI active session. Do not claim Runner session reattachment or per-Agent HAPI availability from CLI-version checks alone; prove those with a later disposable HAPI spawn when credentials and an official Windows executable are available.

# v0.29.0.5 release

The published release fully carries PR #1741's stable Web/iOS/Android `streamId` block identity and the title-provider max-token and timeout controls from the same head. Draft PR #1762 remains deferred. The exact source, release artifacts, and live deployment evidence are recorded in the latest verified release and Windows sections above.

# v0.29.0.4 release

The published release keeps upstream at `980a921ba`, audits 153 open PRs (`53 carry / 96 defer / 4 drop`), and keeps schema V29. Seven new exact-head PRs are included: #1748, #1750, #1754, #1755, #1757, #1760, and #1761; carried #1436 and #1543 were refreshed. #1745 remains deferred because its V25-to-V26 index migration conflicts with maintained schema V29 and does not justify a standalone V30 bump.

The published source tree is `6c14779972d2f5a2da7708957b2df468bf27bdf7`. Bun 1.4.0 builds produced all five CLI targets, and the released darwin-arm64 binary reports HAPI 0.29.0.4. When using `hapi job run`, invoking a Bun 1.4.0 binary is not sufficient if package scripts call `bun` again: prepend the 1.4.0 directory to the job environment's `PATH` and avoid a login shell that rewrites it; verify the compiler path/runtime in build output.

Linux CI confirmed the release candidate, GitHub review found no major issues, and the tag-triggered release workflow produced the signed macOS CLI and desktop bundles. The macOS-local clipboard and serial Runner timing failures also reproduced on the previous exact release and remain classified as baseline test instability, not release regressions.

# Contribution boundary

Searching issues, reproducing problems, and creating an issue are separate from implementing a fix and publishing a PR. Approval to implement a fix does not imply approval to publish it.
