---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-09-15
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

`v0.30.4.1` 於 2026-09-14 發布、2026-09-15 完成 Unix 7 機與 standalone Hub 驗證。官方 pin 是 GitHub Release `v0.30.4` / `8cefb0f04c413d4f47cd364bcf556e03af6b4073`，可為當時 `upstream/main` 的 ancestor，不必等於 tip。維護 SHA / tag / `origin/main` 為 `76d578dcaa6c8dd7cc0d3a6ce56eb2513da397c7`。`cli/package.json` 的 `optionalDependencies` 維持官方 `0.30.4`，不要改成維護版號。

新鮮稽核 156 個 open PR：`27 carry / 125 defer / 4 drop`。Carry 集合與 `v0.29.1.1` 相同；`PERSONAL_PR_POLICY_EXCEPTION` 為 `#1771` `#1635`；`#1320` drop 不 replay。tag 當下的 release notes 仍寫 155/124，以 `pr-audit.tsv` 為準。frozen lockfile 曾因 `tar@7.5.2` 變成 `7.5.22` 秒敗，已重產 `bun.lock`。巨大 PR 上 Swear Review 可能 timeout；此次 latest-head Gemini 0 inline 作為過關證據。

Release Actions `34822868886` 與 tag Test `34822868907` success；8 條 payload checksum 與 macOS `xyz.hapi.cli` / `HAPI Local Release Signing` 通過。核 `checksums.txt` 時不要用子字串 `darwin-arm64`，它會連 `hapi-desktop-darwin-arm64.zip` 一起抓到；應對單一檔名。

Unix 7 機（mazu / cthulhu / athena / valkyrie / zeus / Mac `swairM5` / oracle）binary 與 runner state 均為 `0.30.4.1`；Linux `KillMode=process` 且 `MainPID` 等於 state PID、`NRestarts=0`；Mac launchd `AbandonProcessGroup` / `maxfiles=65536`；oracle PM2 `online` / `treekill=false`。standalone Hub 已離開 9/12 舊 inode，served `index-*.js` 含 `0.30.4.1`，公開與 LAN HTTP 200、`Cache-Control: no-store`，`/cli/upgrade` 無 token 回 401 而非 404。schema `user_version=29`、`quick_check=ok`。Windows `swop` 此輪未部署。

改寫 `origin/main` 用 `--force-with-lease="main:<old-sha>"`；此輪舊 SHA 為 `69ae92524b01bedafe505f2161c54091b425915c`。不要 force-push `upstream`。部署陷阱見 `tools/hapi/supervisor-safe-operations.md`、`tools/hapi/remote-install-stdin-hang.md`、`tools/hapi/hub-sqlite-lock-crash.md`。

先前 `v0.29.0.5` 是從 upstream `980a921ba15665c54998a6ddb658103d467ff4cb` 重建，稽核 154 個 open PR（`54 carry / 96 defer / 4 drop`）。當時 release commit 為 `7a89deefb2cbca900ba54eed1f4e399fada52bb2`，source tree `85ad6564dc84c759099523e80d522fd38df2b371`；PR #16、exact-main review and CI、release workflow `33793933802`、tag test `33793933807`、八條 payload digest 與 macOS signing 通過。當時 8 台 Runner 都驗證到 `0.29.0.5`。Canonical HAPI Skillshare 曾發布於 `shared-skills` commit `913d618fce329ab6832abfce722b56225a3370e2` 與 transfer pointer `7abe2ff5c7ed3ffd077bf843bb144bdae86b7b15`。

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

Do not treat Windows QMD vector embedding as healthy yet. A CPU run remained at zero vectors for about 28 minutes despite consuming CPU, matching upstream Windows hang reports #679 and #739; only that QMD child process was interrupted, leaving the lexical index usable.

The host now resolves Codex `0.152.0`, Claude `2.1.258`, OpenCode `1.18.27`, AGY `1.1.25`, Grok `1.0.5`, and Pi `0.84.4`. Pi uses the renamed official npm package `@earendil-works/pi-coding-agent` and requires Node `>=22.19.0`. Node `24.19.0` and npm `11.17.0` were intact under the versioned WinGet package directory, but neither User nor Machine `PATH` contained that directory; WinGet repair reported that the portable installer technology does not support repair. Adding `<local-appdata>\Microsoft\WinGet\Packages\OpenJS.NodeJS.LTS_Microsoft.Winget.Source_8wekyb3d8bbwe\node-v24.19.0-win-x64` to User `PATH` restored Node, npm, QMD, OpenCode, and Pi command discovery without replacing their existing global package tree. Revalidate this entry after Node upgrades because the path is version-specific.

The Pi fleet uses the same three-layer model restriction: `settings.json` supplies the enabled-model list, `pi-model-filter@0.1.2` applies a default-block filter, and `strict-model-allowlist.ts` enforces the allowlist at runtime. The eight required identifiers are `openai-codex/gpt-5.6-luna`, `openai-codex/gpt-daybreak-blue-latest`, `openai-codex/gpt-5.6-sol`, `openai-codex/gpt-5.6-terra`, `meta/muse-spark-1.2-contributor`, `valkyrie-ninfer/qwen3.8-27b`, and the two `opencode-go` DeepSeek ids, which the 2026-09-11 rename changed from `deepseek-v4-pro`/`deepseek-v4-flash` to `deepseek-pro`/`deepseek-flash` on hosts that completed the rollout; NInfer is the current default. Renaming those two ids must never drop the other six: a host where the allowlist, `model-filter.json` rules, or `enabledModels` contain only the two DeepSeek ids is a regression, and the seven-id `allowed` set plus the `meta/muse-spark-1.2-contributor` `includes` special case in `strict-model-allowlist.ts` is the expected shape. The Pi static OpenAI catalog exposes Luna, Sol, and Terra, while Daybreak Blue needs the custom provider-model entry that maps its public alias to Sol. On 2026-09-04, the HAPI machine `pi-models` API returned exactly these eight effective picker models on all eight hosts after `swop` received its local `opencode-go` gateway. Its existing strict allowlist also retains the canonical `openai-codex/gpt-5.4-mini`, which remains intentionally ineffective while absent from `settings.json` and the default-block filter. Muse requires `pi-meta-oauth` plus Meta OAuth or `META_API_KEY`/`MODEL_API_KEY`; preserve existing `auth.json` when deploying model configuration. Distinguish a real stdout model row from `No models match pattern` on stderr, and verify each host through the HAPI machine API because CLI filtering and strict runtime filtering are separate layers.

Do not treat an exported `transfer_MAC` Pi snapshot as current fleet truth. During this onboarding, that snapshot omitted the deployed `pi-meta-oauth`/Muse state and would have narrowed a live configuration incorrectly. Reconcile the live `settings.json`, filter rules, strict runtime allowlist, installed extension packages, and provider-auth presence before copying or replacing any layer; keep a pre-change backup and verify the final union from live hosts.

Long-lived HAPI sessions can retain a stale process `PATH` after User `PATH` is repaired. Before installing a supposedly missing Node or Pi, check the known versioned WinGet directory and invoke the existing binaries by absolute path. A coordination message delivered to another session also does not cancel a tool call already in flight: after redirecting concurrent work, inspect the exact targets, remove only artifacts proven to have been created by that work, and perform a fresh readback. On 2026-09-04, a stop/handoff instruction arrived after an in-flight fleet operation had already begun; Zeus and Oracle therefore received the missing NInfer strict entry and a subsequent eight-host HAPI readback found both NInfer and Muse on every host. No Runner was restarted. Treat further fleet changes and real Muse/NInfer inference as operator-owned work.

Cursor Agent `2026.09.02-c22c1a3` is now installed through the reviewed official Windows PowerShell installer, and both `agent` and `cursor-agent` resolve. Gemini remains retired. Preserve newer working official versions rather than rolling them back merely to match an older fleet snapshot.

At the onboarding checkpoint, the HAPI launcher and Runner both reported `0.29.0.4`; Hub state recorded active Runner PID `10864`, the expected generation `102c471d67d16c3fdd037a5187ee6b1d45b5c63666f2659eeadb4dc6b41ac40c`, supervised restart, and no last spawn error. The existing Codex session process PID `4116` survived that graceful Runner restart and remained responsive, but the replacement Runner's local session list did not re-adopt it. The later `v0.29.0.5` exact-source handoff did preserve its existing HAPI active session. Do not claim Runner session reattachment or per-Agent HAPI availability from CLI-version checks alone; prove those with a later disposable HAPI spawn when credentials and an official Windows executable are available.

# v0.29.0.5 release

The published release fully carries PR #1741's stable Web/iOS/Android `streamId` block identity and the title-provider max-token and timeout controls from the same head. Draft PR #1762 remains deferred. The exact source, release artifacts, and live deployment evidence are recorded in the latest verified release and Windows sections above.

# v0.29.0.4 release

The published release keeps upstream at `980a921ba`, audits 153 open PRs (`53 carry / 96 defer / 4 drop`), and keeps schema V29. Seven new exact-head PRs are included: #1748, #1750, #1754, #1755, #1757, #1760, and #1761; carried #1436 and #1543 were refreshed. #1745 remains deferred because its V25-to-V26 index migration conflicts with maintained schema V29 and does not justify a standalone V30 bump.

The published source tree is `6c14779972d2f5a2da7708957b2df468bf27bdf7`. Bun 1.4.0 builds produced all five CLI targets, and the released darwin-arm64 binary reports HAPI 0.29.0.4. When using `hapi job run`, invoking a Bun 1.4.0 binary is not sufficient if package scripts call `bun` again: prepend the 1.4.0 directory to the job environment's `PATH` and avoid a login shell that rewrites it; verify the compiler path/runtime in build output.

Linux CI confirmed the release candidate, GitHub review found no major issues, and the tag-triggered release workflow produced the signed macOS CLI and desktop bundles. The macOS-local clipboard and serial Runner timing failures also reproduced on the previous exact release and remain classified as baseline test instability, not release regressions.

# Contribution boundary

Searching issues, reproducing problems, and creating an issue are separate from implementing a fix and publishing a PR. Approval to implement a fix does not imply approval to publish it.

Treat `tiann/hapi` upstream actions as separate authorization boundaries. A request to fix or update the operator's existing PR authorizes commits and pushes only to that PR's existing head branch; it does not authorize merging the PR, enabling auto-merge, closing it, or changing upstream repository state beyond the requested PR update. Green CI, a clean review, and `MERGEABLE` status are evidence that the PR is ready for maintainers, never implied permission to merge. Invoke a merge operation only when the operator explicitly asks to merge that exact PR. This repository-specific boundary overrides the generic `personal-pr-workflow` auto-merge default.

The maintained `swear01/hapi` fork is a separate repository boundary. Updating an upstream contribution does not authorize moving the fork's `main`, rebuilding or publishing a release, or deploying the fleet; each requires its own explicit request or an already-authorized release workflow.

# PR #1607 Delete Group failure

On 2026-09-04, production Hub logs and a rollback-only database reproduction confirmed that group deletion returned HTTP 500 even when every target session was stopped and archived. Bun SQLite counted foreign-key cascade deletions in `Statement.run().changes`, so deleting five direct session rows with existing child data reported a much larger change count and tripped the transaction's exact-count guard. The minimal fix on the existing PR head uses `DELETE ... RETURNING id` and counts only returned session rows; the regression test adds a child message and verifies both sessions and the cascaded message are removed. Commit `f6cb2324fddb8bf3062faa6d077bcd0917311384` passed the upstream PR's `test`, `integration`, `drift-gate`, and latest-head review with no findings. The PR remained open for an upstream maintainer; no merge was authorized.

On 2026-09-14, #1607 head `311e55bb` also closed a separate live-state race: a reconnecting archived session can be active in SessionCache while SQLite still says inactive. Immediately before synchronous transactional bulk deletion, validate both live cache activity/lifecycle and database namespace/archived/inactive state. One invalid member must reject the whole group without deleting histories or emitting removal events. The regression failed before the cache guard and passed afterward; current-head CI and review passed without merging.

Project-group keyboard handlers must ignore events whose target is a child control rather than the group header; otherwise Enter/Space on Copy Path or New Session can be swallowed. Retain full group membership across sidebar search and pinned/hidden rows when applying bulk actions.
