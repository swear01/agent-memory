---
title: Maintained HAPI fork uses an audited upstream overlay and separate contribution permissions
scope: projects/hapi-fork-maintenance
project: hapi
status: active
confidence: high
evidence: Repeated audited rebuilds, rehearsal gates, and explicit issue/fix/PR permission boundaries.
created: 2026-08-18
updated: 2026-10-27
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

`v0.30.7.4` 於 2026-09-28 發布，`main` 與 tag 為 `aec67d5c1ed56f7acccc5965967bdf1bc67a2f6c`，官方 base 為 `86c88df93baf5d1f738dd4b202078bdf6dec376e`。重新稽核 176 個 open PR、carry 25 個：原 24 個中 23 個 head 不變，#1771 更新為 `5a0465f6`，新增 archive 修正 #1930 `86abb6b8`。33 個重產 patches 的 rehearsal tree 相符；相較 `.3` 沒有額外移除功能。已關閉未合併的 #847 不 replay；#1320 僅保留已授權的官方 base 內容。維護記錄位於 `<project-root>/tools/maintenance/releases/v0.30.7.4/`。

最新 head Gemini review 通過；本機完整測試 8,335 passed／16 skipped、實際 Codex/mock-model integration 5 passed。exact-main 與 tag Test、Android、iOS、fixtures、relay CI 通過；iOS 既有 transcript 3 秒等待偶發逾時，於同 SHA 重試一次成功，沒有放寬測試。Release workflow `36348621399` 成功，9 assets、8 payload checksums 通過，兩種 macOS 架構的嚴格簽章與原安裝版 designated requirement 相符。PR `swear01/hapi#28` 已合併；驗證收據見該 PR 的 comment `5859827960`。

Mac、mazu、cthulhu、athena、valkyrie、zeus、Oracle 的 binary／Runner 均為 `.4`，七台 canonical machines active，Hub 與公開 Web asset 也為 `.4`。Mac 與 mazu 的正式 `spawn-peer --agent codex` 新 session 能回覆，首次 archive 成功且讀回 inactive／archived；六個既有 session roots 的 PID 與 start time 維持不變。這次已修好先前 `.3` 尚未修復的 native thread binding 問題，詳見 `tools/hapi/codex-spawn-remit-thread-binding.md`；archive confirmation 是另一個 runner 問題，見 `tools/hapi/session-lifecycle-exit-confirmation.md`。

Windows `swop` 未部署，最後回報仍為 `.2`；從 Mac、mazu、Oracle 探測外部 SSH TCP 22 均逾時，mazu 探測舊 LAN 路由亦逾時。2026-09-28 的結果為 **7/8**，不是全 fleet 完成。未來恢復部署前重新核對當前 `Host swop` 路由與 TCP 22，不以 ping 或 `ssh -G` 當成連線證明。`.3` 起已移除舊 `hub-artifact` 自升級路由，下面 Windows 流程僅為歷史紀錄。

Hub 更新前 backup 與更新後 live DB 的 `quick_check=ok`，schema 保持 27。只清除經程序／FD／maps 查核無使用的 10 個 runtime directories 與四份舊 installer archives，約 1.77 GiB；mazu `.2`／`.3`、zeus `.2` 及仍被 session/helpers 使用的舊 `.nfs*` executable 必須保留，不能為清理而停止既有工作。

Canonical gist、handover mirrors 與 HAPI skill 文件已同步。`shared-skills#43` 與 `transfer_MAC#53` 已合併，七台 source 讀回 `a24779d4db451a9f05b6a1388b481d2f8b888187`、六個 targets／106 links 且 source clean；parent main 為 `b398629b91ca68e9fecceb9d02599cedefc0811b`。原有其他 local edits 保留。三個本輪 release/docs worktrees 已清理，上游 #1930 的 worktree 保留等待 maintainer。

# Historical v0.30.7.3 release

`v0.30.7.3` 於 2026-09-27 發布，SHA `e80ab76f1ab2d9f9ac22a5e115ad9029f0816410`，carry 24 個 open PR，排除 closed/unmerged #847；Release Actions `36292425047`、9 assets／8 checksums 通過。Unix 七機部署 `.3`，Windows `swop` 當時仍 `.2`／離線，使用者暫緩處理。Mac `skillshare sync` 曾遇設定檔 `Operation not permitted`，重試成功。

# Historical v0.30.4.1 release

`v0.30.4.1` 於 2026-09-14 發布、2026-09-15 完成 Unix 7 機與 standalone Hub 驗證。官方 pin 是 GitHub Release `v0.30.4` / `8cefb0f04c413d4f47cd364bcf556e03af6b4073`，可為當時 `upstream/main` 的 ancestor，不必等於 tip。維護 SHA / tag / `origin/main` 為 `76d578dcaa6c8dd7cc0d3a6ce56eb2513da397c7`。`cli/package.json` 的 `optionalDependencies` 維持官方 `0.30.4`，不要改成維護版號。

新鮮稽核 156 個 open PR：`27 carry / 125 defer / 4 drop`。Carry 集合與 `v0.29.1.1` 相同；`PERSONAL_PR_POLICY_EXCEPTION` 為 `#1771` `#1635`；`#1320` drop 不 replay。tag 當下的 release notes 仍寫 155/124，以 `pr-audit.tsv` 為準。frozen lockfile 曾因 `tar@7.5.2` 變成 `7.5.22` 秒敗，已重產 `bun.lock`。巨大 PR 上 Swear Review 可能 timeout；此次 latest-head Gemini 0 inline 作為過關證據。

Release Actions `34822868886` 與 tag Test `34822868907` success；8 條 payload checksum 與 macOS `xyz.hapi.cli` / `HAPI Local Release Signing` 通過。核 `checksums.txt` 時不要用子字串 `darwin-arm64`，它會連 `hapi-desktop-darwin-arm64.zip` 一起抓到；應對單一檔名。

Unix 7 機（mazu / cthulhu / athena / valkyrie / zeus / Mac `swairM5` / oracle）binary 與 runner state 均為 `0.30.4.1`；Linux `KillMode=process` 且 `MainPID` 等於 state PID、`NRestarts=0`；Mac launchd `AbandonProcessGroup` / `maxfiles=65536`；oracle PM2 `online` / `treekill=false`。standalone Hub 已離開 9/12 舊 inode，served `index-*.js` 含 `0.30.4.1`，公開與 LAN HTTP 200、`Cache-Control: no-store`，`/cli/upgrade` 無 token 回 401 而非 404。schema `user_version=29`、`quick_check=ok`。Windows `swop` 此輪未部署。

改寫 `origin/main` 用 `--force-with-lease="main:<old-sha>"`；此輪舊 SHA 為 `69ae92524b01bedafe505f2161c54091b425915c`。不要 force-push `upstream`。部署陷阱見 `tools/hapi/supervisor-safe-operations.md`、`tools/hapi/remote-install-stdin-hang.md`、`tools/hapi/hub-sqlite-lock-crash.md`。

先前 `v0.29.0.5` 是從 upstream `980a921ba15665c54998a6ddb658103d467ff4cb` 重建，稽核 154 個 open PR（`54 carry / 96 defer / 4 drop`）。當時 release commit 為 `7a89deefb2cbca900ba54eed1f4e399fada52bb2`，source tree `85ad6564dc84c759099523e80d522fd38df2b371`；PR #16、exact-main review and CI、release workflow `33793933802`、tag test `33793933807`、八條 payload digest 與 macOS signing 通過。當時 8 台 Runner 都驗證到 `0.29.0.5`。Canonical HAPI Skillshare 曾發布於 `shared-skills` commit `913d618fce329ab6832abfce722b56225a3370e2` 與 transfer pointer `7abe2ff5c7ed3ffd077bf843bb144bdae86b7b15`。

The `v0.29.0.5` app shell returned no `Cache-Control: no-store` header through either the Hub-local or public route during rollout verification. Treat this as an unresolved Web-cache observation, not as a failed Runner deployment or a confirmed regression, until compared with the intended header contract and a previous release.

Binary replacement can briefly remove `runner.state.json` while a supervised Runner completes its own handoff. Treat a missing state file during this window as transient: re-read supervisor and state together before taking action. If ownership remains split, stop the supervisor, terminate only the exact state PID after verifying it is `hapi runner start-sync` without `--started-by runner`, then start the supervisor and require its PID to match the new state PID.

# Historical Windows Runner self-upgrade (before v0.30.7.3)

At the time, the Windows `swop` Runner had no known direct SSH management path. A standalone compiled Hub with `HAPI_UPGRADE_CHANNEL=off` also could not materialize a `hub-artifact` by itself because it had no monorepo root. The then-verified narrow upgrade path was:

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

# v0.30.7.1 release

- Historical policy at this release: PRs authored by `swear01` were auto-carried. Superseded by `v0.30.7.3`: closed and unmerged PRs are excluded regardless of author.
- PR #1436 (`feat/web-persist-voice-input-session-switch`) rebased onto upstream `main`, tests passed, and pushed to `swear01/hapi`. Restores the voice dictation direct-send button (`dictationCanDirectSend`) in `HappyComposer.tsx` and `ComposerButtons.tsx`.
- Upstream release lineage: `sync-from-upstream.sh` updated so that release tags whose parent commit is an ancestor of `upstream/main` (e.g. `v0.30.7` at `0239edf38`) are accepted.
- Published release `v0.30.7.1` on `swear01/hapi` with 9 release assets and sha256 checksums.
- Deployed across the fleet (`mazu`, `cthulhu`, `athena`, `valkyrie`, `zeus`, `oracle`, `mac`). Preserved all running child sessions across systemd (`KillMode=process`), macOS launchd (`AbandonProcessGroup=true`), and PM2 (`--no-treekill`).

# v0.30.7.2 release

- 修復 Web 對話滑動時 React 崩潰：`Maximum update depth exceeded. The result of getSnapshot should be cached to avoid an infinite loop.`。
- 根因分析：
  - `@assistant-ui/react@0.14.29` 宣告之依賴 `"@assistant-ui/tap": "^0.9.6"` 在構建 `v0.30.7.1` 更新 lockfile 時自動解析漂移至 `0.9.18`。
  - `tap >= 0.9.14` 引入了嚴格的 React 18 `useSyncExternalStore` snapshot 引用一致性檢查（PR #5897）。
  - 而被搭配的 `@assistant-ui/core@0.2.23` 屬於已凍結的 0.14/0.2 系列，未被 backport 該修復。其 `ThreadListRuntimeImpl.subscribe` 錯誤訂閱至 `_core` 而非 `_stateBinding`，導致 `LazyMemoizeSubject.getState()` 在未連接狀態下每次皆回傳新建的 `{ mainThreadId }` 物件。
  - 官方 `v0.30.7` 預編譯二進位因鎖定在 `tap@0.9.8` 故暫時未爆發，但任何從原始碼安裝或 lockfile 更新者皆會受影響。
  - 觸發時機為上游 PR #1809（冷啟動 20 則小分頁與滾動 coverage 載入），使用者滑動時頻繁 re-render 撞上 snapshot 死循環。
- 補丁修復：透過 pnpm patch 建立 `@assistant-ui/core` 補丁（針對 0.2.23 版），修正訂閱對象並在 `LazyMemoizeSubject.getState()` 加入 `shallowEqual` 快取。
- 自動化驗證：Playwright 對線上 Production Session 進行 40 次快速滾動測試，確認 0 page errors。
- 部署驗證：發布 `v0.30.7.2`（9 項 release assets），平滑滾動更新全 fleet 8 節點（`mazu`, `cthulhu`, `athena`, `valkyrie`, `zeus`, `oracle`, `mac`, `swop`）與 `mazu-hub`，100% 保留所有運作中工作 session。

## 2026-10-08：更新既有 upstream PR 的已驗證陷阱

- GitHub PR 的 `base.ref=main` 不保證 `base.sha` 是最新 main。此次 #1542、#1662 的 SHA 仍指向 9 月舊版本；新的審查工作流依該 SHA checkout trusted policy，因當時還沒有 `.github/scripts/pr-review.cjs` 而失敗。對同一 PR 執行 `PATCH /repos/tiann/hapi/pulls/<number>`、body `{"base":"main"}`，可在不改目標分支或 head SHA 的情況下刷新 base SHA。相同分支的刷新沒有自行啟動新 review；更新具體維護說明才觸發 `edited`。刷新後兩個 review 都通過 prepare 階段；模型通道 503 是另一個外部服務故障，不能當成程式碼 finding 或 clean review。
- 以新的 upstream-main worktree merge 既有 PR head，可保留原 head 為 ancestor，並 fast-forward push 回原分支，不必 force push。推送前同時核對 PR head 與遠端 branch SHA；既有使用者 worktree 不跟著 reset。已 staged 的刪除路徑不應再次列入 `git add`，只 stage 尚未 staged 的變更。
- #1822 的 Cursor ACP wire/effort 實作已經由合併的 #1819 納入 main。直接重整舊 PR 會留下重複 `findCatalogBaseKey`，造成 typecheck 失敗；先檢查已合併 PR 的原始 head 與最終程式碼，再判斷是否過時。
- #1436 的失敗語音傳送 recovery 必須區分「沒有 live composer」與「使用者已把 live composer 清成空字串」。後者不能回退到 debounce 尚未更新的 persisted draft，否則已刪除的 follow-up 會復活。使用真實 `useComposerDraft` 的 hydrate → clear → late failure → unmount regression 已驗證此差別；standard/realtime 共用 `recoverFailedVoiceSend`。
- 大量訊息的 browser fixture 在受限 CPU 上可能因 render 超過 polling budget 而失敗，即使 trace 最終讀到正確 row count。保持功能 assertions，給足有證據支持的等待時間，並在足量 ingests 後停掉有限驗證用 timer 再測 viewport；另一個 continuous-streaming scenario 仍需保留。未修改 upstream 的同一 scrollbar assertion 也失敗時，應記錄 matching baseline，不能把它當成 feature regression。

## iOS cell recycling：先查維護版現有 patch

PR #1771 在 upstream `99cd3d417` 上的 iOS job `113507830463`，於 `ToolTranscriptPresentationTests.testPlanPublicationUpdatesRecyclingAndInspectorPreserveTheDocumentAndAnchor` 的 cell 回收斷言失敗。相同測試已在維護版 `v0.30.7.4` 的 `tools/maintenance/releases/v0.30.7.4/patches/0032-test-ios-wait-for-transcript-cell-recycling.patch` 修正：捲走後等 `cellForItem == nil`，捲回後等 `cellForItem != nil`，取代固定 150ms／200ms sleep，保留回收、高度與閱讀位置 assertions。遇到此失敗時，先比對 audited patch queue，避免重做已有修法。這是可重用的候選修法，不代表尚未套用它的 PR 已通過原生 CI。

## 2026-10-12：審查修復的可重用判斷

- ElevenLabs React SDK `0.13.0` 的 `useScribe` 有自己的 unmount effect，會無條件 `connectionRef.current?.close()`。HAPI hook 即使在 pending-send cleanup 提早 return，也阻止不了 SDK 的 cleanup。PR #1436 改由現有 client `Scribe` 連線管理 finalization 壽命：最終 transcript／有限逾時／錯誤後關閉，普通未送出的錄音仍在卸載時取消。直接宣告已鎖定的 `@elevenlabs/client@0.13.0` 即可，不需升級 SDK。延遲 commit 在 unmount 後的 regression 先失敗再通過；逾時與普通卸載也有覆蓋，`3cc695e93` 的功能 CI 與最新 HAPI Bot review 通過。這些證據沒有包含真實供應商語音連線。
- Copilot／Kimi 的 advertised effort 不保證 `session/set_config_option` 會接受。僅識別本地 unsupported/unavailable 字串的 startup catch，仍會把供應商 `Invalid params` 拋出、在 prompt dispatch 前結束 launcher；queued-effort 的成功回復測試無法驗證 startup。兩條路徑都需測試，啟動拒絕時回報錯誤並發布 discovered effort，再繼續處理 prompt。Cursor Auto 的 toolbar label 正確也不代表 picker row 正確；兩者需共用 flavor-aware normalization，並覆蓋 `auto`／`default`／`default[]` 的選取 indicator。
- UIKit 自 iOS 15 起，`UICollectionView.cellForItem(at:)` 可回傳尚未顯示、留在 prepared-cell cache 的非可見 cell。單靠加長等待不能證明 cell 已回收。若測試目標是強制驗證回收／重建，僅在該 fixture 關閉 `isPrefetchingEnabled`，再保留 nil／重建、高度與閱讀 anchor assertions；產品預取不需改動。模擬捲走時也要像捲回一樣通知 `scrollViewWillBeginDragging`，否則 renderer 保存的 reading anchor 可能在 snapshot 更新後把 viewport 捲回，讓 nil 等待一直失敗。閱讀寬度改變時，bounds 更新不代表 self-sizing 完成，應等實際 cell 的寬度、置中與 anchor 一起穩定，不能只延長固定 sleep 或放寬誤差。
- `TranscriptFlowLayout` 的 preferred-height invalidation，在寬度／字體改變、重置 measurements 後，不能拿 UIKit 留存的 `originalAttributes.frame` 判斷 row 是在 viewport 上方或部分可見。先 `prepare()` 並使用目前 indexed `frames[index]`，才能計算正確 content-offset compensation。新的 regression 故意傳入舊高度 attrs，期待 delta -50 與 anchor -31；舊公式會得到 -94，修正後原生 iPhone suite 通過。保留一點的閱讀位置斷言，等待並不能修正實際漂移。
- Session 的 process-only stop 不能因 hub lease 已過期就跳過 CLI 的 `archive:false` intent，也不能在收到 intent 後仍按 session ID 強殺 runner process：SIGTERM cleanup 仍可能寫 archive metadata。先取得 CLI acknowledgment 及其 PID，再只用 raw PID probe 等待退出；目前 runner contract 在缺 start marker 時只觀察、不殺 process。CLI 不可達時 fail closed；明確 delete 的路徑則先使用 archive intent，再刪 cache。regression 覆蓋過期 lease、ack PID 與 metadata PID 不同、退出延遲及 metadata 保持 running。

## 2026-10-14：Composer 版面同步與冷載入測試

- 上游 #2007 將 primary `UnifiedButton` 從 `ComposerButtons` 移到 `HappyComposer` 的側邊欄。同步 #1436 時，不能只在 toolbar 傳 `dictationEnabled`／`dictationCanDirectSend`；側邊 primary 和 toolbar 的 compact fallback 都要保留傳遞，才能讓錄音中的直接 Send 繼續走 session-bound finalization，同時保留大尺寸 Stop。合併兩組按鈕測試，再加同時點 Send／Stop 的 fill-height regression；56 個 focused tests 通過。#1418 的 drop callback 仍要接在 `RichComposerInput`，不能跟著錯位的 conflict context 接到 expanded textarea；5 個實際 HappyComposer browser drag scenarios 通過。
- 多個 worktree 的完整 gate 在 2 CPU 機器並行時，`claudeRemoteLauncher.launchFailure.test.ts` 首個 case 會把大型 dynamic import 算進預設 5 秒期限。暫時量測確認 import 約 6.6 秒、實際 launcher 約 50ms；診斷修改還原後，原 assertion 在較長的本機執行期限下通過。先減少並行與區分載入／執行時間，不要把這個逾時判成 queue recovery regression 或改動產品重試行為；CI 保留原測試期限。

- Pending 語音 Send 不能只用文字相等判斷 draft 是否仍是送出前的 baseline。使用者清掉 composer 再輸入完全相同的 `continue`，即使 debounce 尚未更新 storage，也屬於新的 follow-up。#1436 為每個 session 記錄有界的遞增 revision，live composer edit、persist／clear 都更新；standard／realtime 的成功清理、resume transfer、失敗 recovery 一起比對文字與 revision。被淘汰的 revision 必須回傳新的全域序號，避免舊 pending-send token 再次匹配。真實 `useComposerDraft` 的 10 個成功／失敗、同 session／resume、live／unmount scenarios 及 Web 全套 3,414 tests 通過；storage 仍是原來的 plain-string map。
- Bun compiled CLI 的 `process.cwd()` 會解析 symlink 成實體路徑；fresh session selection 若以 runner 請求的 logical directory 精確比對 metadata.path，會拒絕有效 child。#1771 在 `spawnHappyCLI` compiled branch 有明確 `options.cwd` 時同步設定既有 `HAPI_INVOKED_CWD`，保留沒有 cwd 時的環境繼承；不需放寬 Hub directory guard 或新增 wire field。實際 Bun 子程序的 symlink regression 在修正前回報 physical path、修正後保留 logical path，CLI typecheck 與全套 3,047 tests 通過。

- #1436 的 inactive composer 在 hidden attachment hydration 尚未完成時，`blocksScheduling` 為 true；空草稿載入完成後變成 false。若 `handleSend` 的 `useCallback` 漏掉這個 dependency，Send 的顯示已解鎖、callback 卻仍只 stop/transcribe 而不發送。整合 regression 要使用穩定的 assistant-ui API identity，且 mock button 像正式元件一樣呼叫 `onSend()`／`onSend('default')`，不能把 click event 當 send intent 傳入。載入前後 eligibility 與 `stopAndSend` 都有斷言，dependency 修正後 45 個 composer/button tests 與 Web typecheck 通過。
- #1771 的 process-only stop 不可把 terminal session 的 `active:false` 當已退出；它可能只是 SessionCache lease 過期。已離線的 terminal 和 runner session 都先嘗試 non-archiving RPC，再用已回報／記錄 PID 的唯讀 Runner probe 確認 `already_gone`；`unknown`／`still_alive` 或缺少 process evidence 必須拒絕。只有原本 active 的 terminal 在成功 acknowledgment 之後等到新的 session-end，才能沿用無 Runner 的 fast path。5 個 terminal regressions 在修正前全失敗，修正後 Hub typecheck 與全套 1,443 tests 通過；不存在的 session 仍保持 idempotent。

- Background voice finalization 的 session resolver 不能捕捉 render-time `session.active`。錄音當下在線、final transcript 到達前離線時，舊 resolver 會略過 resume，最後 POST 得到 `session_inactive`。#1436 的 voice-only resolver 在 delivery 前 `api.getSession`，在線直接使用、離線以最新 permissionMode 呼叫既有 `resumeSession`；文字與 upload resolver 不需跟著改。61 個相關 tests 與 Web typecheck 通過，包含真實 resolver 接到 delayed Scribe commit、先 unmount 再離線、最後恰好一次送到 resumed ID 的情境；lookup／resume 失敗沿用 transcript recovery。
- 上游 Issue #1974 的 `ping_peer` 缺少 localId 問題，不能當作使用者 fork 中全新的缺陷：既有 #1771 和 audited `v0.30.7.4` 都已在 `cli/src/modules/pingPeer/pingPeer.ts` 用 `providedRemitId ?? randomUUID()` 產生穩定 remit UUID，POST 帶 `localId: remitId`。官方缺少的是獨立、容易合併的小修法；後續可以拆出 sender 的最小 patch 與 regression，避免在 fork 重做同一修復。推薦 issue 前要同時查 upstream main、既有個人 PR 和實際 release tree。

## 2026-10-15：Codex 回合識別、Peer 去重與 Cursor／Pi 補修

- Cursor ACP 原地切換至 Auto 期間，backend 暫時為 null；這時啟用 Auto-review，內部 `/auto-review` 如果沿用 `session.model` 舊快照，重啟後 dispatch 會把 Auto 改回舊模型。這是 permission-only 控制訊息，不應附帶 model intent；保留 permissionMode，移除該筆內部訊息的 model。Regression 需等到 slash command 真正送達並斷言最後 `session.model === 'auto'`，只確認 enqueue 或 picker label 不足。補修已在原作者 feature head 與目前 upstream 上通過完整 gate；貢獻 PR `heavygee/hapi#272` 的 commit `11721f36c` 通過最新 Codex App review。Fork 沒啟動 Actions 不代表 CI 通過，必須另外讀 source/merge SHA 的 runs/checks。
- Pi RPC 的 assistant delta 先排入 `publishChain`；同步 prompt rejection 或 transport cleanup 若直接 flush accumulator，會早於排隊 delta，造成最後一段內容遺失。把 flush 排入相同 promise chain，正常 cleanup 需 await；新增 rejection／transport regression 要先證明原始 production 失敗。圖片本地讀取的 `stat(path)`→`readFile(path)` 有路徑替換與檔案增長競爭，應單次 open、在同一 FD fstat、按已驗證上限有界讀取，最後再檢查 size 並保證 close；使用可用的 nonblocking open 避免 FIFO/device 卡住。PNG magic 必須完整八 bytes，否則上游工具接受後仍可能被既有轉換器丟棄。
- 給另一位作者現有 feature PR 做小補修時，publication diff 必須以該作者的實際 feature head 為 base；把已合併目前 upstream 的驗證分支直接投向舊 feature branch，會夾帶大量無關 commits。可保留 upstream-integrated worktree 做相容性驗證，再從 feature head 建立獨立補修 worktree、重跑該實際 publication tree 的完整 gate，保留原作者歷史。Clone-local upstream-only hook 若不適用，必須保留 task worktree、實際 target base、精確檔案 scope 與 head guard 的等價檢查，不能全域關閉 hook 或用 no-verify 偽造過關。

- Foreign fork 的 `statusCheckRollup` 可能完全漏掉 `action_required` 的 Test run，因為尚未建立任何 job/check；必須另查 exact source SHA 的 Actions runs。Fork 原先 workflow registry 沒列出審查，不保證 PR 開啟後仍沒有：新 workflow 可能這時才註冊。Pi 補修 `luodeb/hapi#1` 的 Test 等 owner approval，review 在 Codex 執行前因 proxy server-info JSON 缺失而失敗，不能標成 code-test failure 或 clean review；先保留可審核 PR／本機驗證，讓 fork owner 處理核准與缺少的 review configuration。

- 向另一位作者的 feature branch 提交補修 PR，不會直接改動其 branch，也不會更新該作者對官方的原 PR；只有作者接受補修後才會傳遞。回報時分開標示官方 PR、作者 fork 的補修 PR 與已發布維護版。Pi #1830 的主功能已有官方 #1831，不能稱為無實作 issue；2026-10-17 重新確認原 head `0ab08bfc` 的最新審查仍有 queued-text loss 與 unbounded-read finding，CI 綠色僅證明現有測試通過，不表示審查無缺陷。補修目的應說明為修復已重現的缺陷，並揭露作者接受與 fork CI 核准的額外等待。

- 多個 worktree 的 CLI Vitest 若共用預設暫存目錄，`globalSetup.ts` 的固定 `hapi-test-config.json` 會互相覆寫／被另一個 teardown 移除。2026-10-17 的 #2025 gate 在 7 files 通過後，另 256 files 因設定檔遺失而無法 setup；當時觀察到其他任務同時跑 CLI，但沒有直接觀察到歷史 unlink PID，不能指名是哪個程序刪除。沿用現有 TMPDIR 傳遞、給整個 gate 一個短的 task-specific directory，再完整重跑即可隔離：四套 typechecks、8,188 tests 通過，包括原先 setup 被擋的全部檔案。無需更改產品、跳過測試或終止其他任務。此 workaround 不等於已修正共用暫存檔的測試設計。

- 原生 CI 的第一個失敗會遮蔽後續 stage。修 #2025 時，只移植 #1771 的 cell-recycling fixture 修補後，iPhone 193 tests／7 skipped／0 failures，但原先被擋住的 iPad stage 才暴露 `TranscriptTypographyTests.testReadingWidthAndBoldTextChangesKeepHistoryInPlace` 的 anchor 漂移。還需移植 `AnchoredTranscriptList.swift` 的 current-frame compensation、直接 stale-attributes regression，以及等待實際寬度／置中／anchor 穩定的 typography fixture；不能只加 sleep 或放寬一點誤差。檢查第一個紅燈時，同時查 downstream skipped stages 與既有相關原生修補，避免把第一階段修好當成完整成功。最終 `cefea0cfb364c2b7f7d9d24deee3b2166277558e` 的全部十項 CI 與最新 HAPI Bot review 通過；此次仍僅更新貢獻 PR，未合併或發布維護版。

- #1974 已從既有大 PR／維護版拆成只動共用 sender 與測試的 upstream `tiann/hapi#2024`，head `110d381cac8661dda1290089ee186567e43d3b22` 的四項 CI 與 HAPI Bot requirement／approach／code review 全數通過。每次 intentional send 生成新 UUID localId，既讓原始儲存與 native queue mirror 使用同一筆 row，也保留相同文字連送兩次的語義；不能以內容 hash 去重。實際 CLI source entry 對隔離 HTTP server 的兩次登入／傳訊、一個 inactive resume，再經真實 Hub SQLite native mirror，恰好留下兩筆 row。這是預防新重複，不是清理歷史資料，也沒有證據表示原 bug 讓模型執行兩次。

- Codex 回合耗時應使用獨立 `turn_duration` metadata，避免重送 `token_count` 造成 usage 重複計算。`CodexRoundDuration` 只接受已觀察到的 live start／matching completed turn，重播、完成訊息單獨到達、失敗或中止不能啟動計時。Local transcript 使用原始 record timestamp，避免同批掃描得到假的零秒；remote/shared 要在進入 async projection queue 前捕捉 receipt time。持久化尾頁可能不含起始 user row，UI 需從 metadata 還原，而非依賴仍載入的 user timestamp。
- Duration／usage 必須綁定 native turn ID；除一般 transcript／remote/shared messages，也要查直接發送的 Code Mode hooks。完成後 clock 已清除，converter 仍須保留 raw payload 的 turn_id，才能關聯 late token_count。針對回應沒有 turn ID、usage 原本卻有 ID 的既有資料，不能讓新 matching 丟掉原本 Round usage/model；只有完全未標記的 latest group 可保留原有 usage attachment，duration 不得猜配。不同 native turn 共用一個 response group 時，晚到的舊 duration／usage 不能覆寫較新 turn 的摘要。兩個 regression 在修正前確實失敗，修正後相關 Web tests 通過。
- Round summary 可能掛在 response group 的第一張 `update_plan` 卡片。Remote／Shared 若跨回合沿用 `codex-plan-state`，`ensureToolBlock` 會取回舊位置的卡片；新增 turn metadata 時直接重標並清掉 summary，會讓上一回合已保存的耗時與 token/model 摘要消失。兩個 live producers 的 call／result IDs 都需改成 `codex-plan-state:<native-turn-id>`，同回合保持穩定、跨回合分開。兩個 producer regressions 先紅再綠，另驗證兩張 Plan 卡片各自保留摘要。Local proposed-plan 已使用 item-unique IDs；Web/iOS/Android 的 raw `plan_update` legacy normalizer 雖仍有舊 ID，但目前沒有 live producer 送這條路徑，不需為這次補修改寫三份 normalizer。

## 2026-10-18：Pi 圖片功能改投官方與 Pi 1.x 相容性

- 使用者明確決定以目前 upstream/main 重做獨立官方 PR，引用原作者 #1831、保留 Co-authored-by，再整合補修；新官方 #2060 以 `317e39a0b` 為 base、單一 commit `3d9ebb3e04f04baffe3702c2479776eb05330e0d`。這個決定取代等待作者接受 foreign-fork supplement 的路徑，並不授權關閉別人的 #1831、合併官方 PR、更新維護版 main 或部署。 #2060 的四項 CI 與該 head 的 HAPI Bot requirement／approach／code review 全數通過；只關閉自己的過時補修 `luodeb/hapi#1`，原作者 #1831 保留 open。
- `registerGeneratedImageFromAcpBlock` 原本宣告 async，但實際只有有界 base64 decode、MIME sniff 和記憶體 registry；拿掉 async 後 Pi converter 可以保持同步，直接避免圖片功能引入的 publication queue、prompt rejection flush 與 shutdown 排序問題。唯一原有 production caller ACP 仍可 await 普通回傳值；Pi／ACP／registry 的 398 個 focused tests 與完整 8,176 tests、四套 typechecks 通過。不要為純 CPU helper 新增無必要的非同步生命週期。
- Pi 在 2026-10-01 發布 1.0，10-15 最新 1.3；canonical repo/package 已是 `earendil-works/pi`／`@earendil-works/pi-coding-agent`。查文件時歷史 execute signature 可能混入檢索結果，應回到 pinned tag source；v1.3 execute 仍是 `(toolCallId, params, signal, onUpdate, ctx)`，loader 保留 `@sinclair/typebox` alias。隔離實際 Pi 1.3 RPC、unmodified extension、text-only loopback model 已證明 direct image、builtin Codemode nested image、missing-file isError 全通過；nested child ID 為 parent 加 `/1`，圖片在 RPC 保留完整 bytes，即使非視覺模型收到零 image parts。Converter 要保留 failed status 及 image-bearing results，不需要新增 protocol field 或全域升級 Pi。

## 2026-10-20：v0.30.7.5 發版與部署的已驗證陷阱

- Fork Actions 可能被 GitHub 單獨停用、卻看不出來：`swear01/hapi` 從 10/08 起所有 dispatch 都回 HTTP 422 `Actions has been disabled for this repository`，但 `GET /actions/permissions` 仍是 `enabled: true`、Settings 顯示 Allow all，其他 repo 正常；用 API 重存 permission 無效，最後由使用者恢復。發版前先 `gh workflow run fixtures.yml --ref <branch>` 探測，不要等永遠不會開始的 CI。恢復後 close/reopen PR 沒有觸發 `pull_request` Test；推一個指向同 SHA 的暫時分支（`<sha>:refs/heads/ci/<tag>`）可觸發 push-event Test，用完刪掉。Android/iOS 只在 PR 或 main（path filter）執行，所以 native gate 落在 exact-main push。
- 上游 main 可能一夜之間合併已 carry 的 PR：#2026 被 squash 進 main，PR 頁面卻仍顯示 OPEN。依政策 rebase 到新 base，manifest 記為 `integrated`（audit decision 仍是 carry），不重放 patch。原本 defer 的 #2041 進了 base 後，與 #1771 刪除 `sessionSummaryInstruction.ts` 衝突：保留刪除，#2041 的 converter 過濾由 base 提供。#2041 新增的 projection 測試要求 history replay 輸出等於 live；#1771 在 replay 已完成回合時會合成 `turn_complete`，所以 live 那半也要送 `turn/completed`，而不是放寬比對。
- 多個 carry 疊在一起時，常見的壞法是測試 harness 跟不上別人新增的 export／method：手刻的 `SyncEngine.prototype.X.call({...})` harness 缺 #2032 新增的 `releaseSessionEnumerations`；`vi.mock` 工廠缺上游新 export（`getTelegramWebApp`、`isComposerPrimaryFillHeight`）。用 `importOriginal` 展開原模組再覆寫，比逐一補 export 穩。修補 fold 進引入該測試的 carry commit（`fixup!` + `GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash`），patch queue 才能重放出同一棵 tree。
- 從 oracle 部署會遇到的限制：oracle 連 mazu 公開 IP 的 SSH 逾時，Mac 沒開 sshd（Tailscale ping 通但 22 port refused）。`hapi-ctl.sh` 是以 Mac 為中心設計的，在 oracle 上跑 `status mac` 讀到的其實是 oracle 本機的 binary。可行做法：`hapi spawn-peer --machine <mac-machine-id> --dir <mac-home> --agent claude`，讓 Mac 上的 session 依 skill 逐台更新。加 `--permission-mode bypassPermissions` 時 spawn 回 HTTP 502；不帶 permission mode 就成功。mazu hub 重啟期間 `wait-peer` 也會短暫 502，用同一 remit ID 重試即可。
- v0.30.7.5 部署結果：6 台可達主機（mazu、cthulhu、valkyrie、zeus、oracle、Mac）都升到 `.5`，baseline session 全部存活；valkyrie 與 Mac 走了 handoff-loop 的 exact-PID recovery；Athena 依政策排除，swop 離線待補。Retention 清掉約 1.02 GiB；NFS 上被替換的 binary 會變成 silly-renamed 的 `.nfs*` 檔案，檢查 live process 時要比對 exe/maps/fd/cwd，不能只看路徑字串。
- 發版 PR 的審查：Cursor Bugbot 會直接拒審超大的 release diff（約 58k 行）；Gemini 在 exact head 的 clean review 就構成 OR gate 的通過。fork 的 `Codex PR Review` workflow 因 runner 上缺 Codex server-info 檔而失敗，與程式碼無關。Codex 的 P2 意見（#1616 sub-path 通知路由、#2032 已封存 session 的 cache entry、#2061 pending 時的 dialog dismiss）都在 carry 的上游程式碼裡，記錄下來、不在維護版自行修補。

## 2026-10-22：v0.30.7.6 候選（#2008 overlay）的已驗證陷阱

- `sync-from-upstream.sh` 對 manifest 的 carry 有兩個硬性檢查。非 owner 的 PR 只要 bot 有 finding，就會報 `PR_QUALITY_GATE_FAILED`。每個 open PR 的 audit head 都必須等於 live head，否則報 `PR_AUDIT_HEAD_STALE`。所以「使用者指定要帶、但 review 有 finding」的 PR，以及「新 head 有衝突、只能留在舊 head」的 PR，都不能記成 carry。做法是 manifest 用 `regenerate overlay`（新 patch 的 source 寫 `-`；沿用舊 fork commit 的話 source 寫那個 commit），pr-audit 的 decision 寫 `overlay`，reason 寫明原因。腳本不限制 decision 的值。
- 若 upstream base 沒變，最快的重建方式：從上一版 origin/main 的最後一個 carry commit 開分支，用 `git merge --squash refs/pr/N` 逐一加入新 carry，用 `fixup!` + autosquash 刷新舊 carry，再 cherry-pick overlay 和 prepare/audit commit。注意多參數的 cherry-pick 會依日期排序，不是依你給的順序。
- 衝突別用 `git merge-file --union`：遇到雙方都新增 describe 區塊的情況，它會吃掉共用的收尾 `});`，也會把雙方的 `it(...)` 開頭拼在一起。正確做法是以 ours 為底，把 base→theirs 的 diff 用 `patch` 套上去，失敗的 hunk 手動補。
- Carry 之間的編譯期衝突要跑 typecheck 才抓得到：#2078 刪了 `webhookTimeoutMs`，但已 carry 的 #2066 還在用。
- desktop workspace 是 overlay commit 才加進來的。若在加入它之前就跑了 `bun install`，之後要重跑，否則 `typecheck:desktop` 會報 `tsc: command not found`（exit 127），整個 parallel typecheck 會被 SIGINT 中止。
- 在 oracle（2 CPU）上，`test:cli:integration` 的 `beforeEach` 等 runner 啟動只給 10s，常常逾時。未修改的 v0.30.7.5 在同一台機器上 13 個 case 失敗，所以這是主機問題，不是 regression。composer-copy e2e 的第一個 case 冷載入要約 31s，要加 `--timeout=120000`。

### v0.30.7.6 Swear Review 處理（2026-10-22）
- Release PR（fork #30）的 Swear Review 會審整個 diff 對 `main`（7 萬多行）。分類方式：對 `git diff --shortstat origin/main HEAD -- <file>` 沒變的檔案，發現是 .5 就有的，不擋這版。
- 對 carry 進來的上游 PR 找到真 bug 時，不改 carry（live head 必須對得上，否則 `PR_AUDIT_HEAD_STALE`）。改成在 carry 後面加一個 `regenerate overlay -` 的 fork fix commit，例如 0056 `fix(codex)` 修 #2092。
- 只有 carry 某個上游 PR 才會出現的 bug（上游 main 沒有），不要另開上游 issue 或 PR。要修的是造成 bug 的那個 PR：如果是自己的 PR 就直接更新；如果是別人的，就做成 fork overlay fix，並在原 PR 留建議。2026-10-25 使用者因此要求關閉 #2154（#1948 的舊模型收合）。
- 重建 patch queue：`git format-patch -q --zero-commit --no-signature -o <dir> <upstream_sha>..HEAD`（不含 record commit）。manifest 後面的列重新編號；`expected_source_tree` 用 HEAD 的 tree 扣掉 release record 目錄後的 write-tree。
- 要合進 tail commit 時，不要用 interactive rebase：先 `git diff` 出成 patch → `checkout -B` 到該 commit → apply + amend → 再 `git cherry-pick A..B`。cherry-pick 不支援 `-q`。
- v0.30.7.6 第二輪：rehearsal 因 `PR_AUDIT_HEAD_STALE`（上游 defer PR 推了新 head）失敗。重跑前先比對所有 open PR 的 live head 與 pr-audit，同時抓「新開的個人 PR」（自動 carry）。這次多 carry 了 7 個 swear01 PR（#2095–#2102）。
- Fork 和上游的已知分歧：peer 指令只吃 exact UUID（沒有 prefix resolver）；Codex permission catalog 多一個 `autoReview`（#2090）；Codex projection 的 history 會送 `turn/completed`（#2025）。合併上游 PR 時，它們的測試期望常要依這些差異調整。

### v0.30.7.6 第三輪：重新盤點（2026-10-23）
- 使用者重新挑了 26 個 PR。非 owner 的 PR 一律走 `regenerate overlay -`，pr-audit decision 寫 `overlay`，reason 開頭用 `OPERATOR_REQUESTED_OVERLAY`，並列出沒過的 gate。swear01 的 PR 照常記 `carry`，rehearsal 會印 `PERSONAL_PR_POLICY_EXCEPTION`，這是允許的。點名要帶、但 fork 早已涵蓋的 PR（#1875、#1869），pr-audit 記 `drop`，寫明涵蓋它的 fork 程式碼，不放進 manifest。
- Rehearsal 也要求 open PR 集合完全一致，不一致時報 `OPEN_PR_AUDIT_SET_MISMATCH`，而且一天內上游就可能新開 20 多個 PR。重跑前用 REST `pulls?state=open&per_page=100 --paginate` 一次拉 live 清單，和 pr-audit 比對 new／closed／stale 三類。新 PR 記 `defer`，也要把它們加進 audit.md 的 open PR 清單。
- Stacked PR（#2056 疊在 #2052 上、#1597 疊在 #1542 上）：用 `git diff upstream/pr/BASE upstream/pr/N | git apply -3`。若碰到 fork 已刪除的檔案，加 `--exclude='*systemPrompt*'` 之類的 pattern，否則整包會被拒。
- Fork 的結構性分歧（合上游 PR 時常要改寫）：
  - 沒有 peer system prompt，也沒有 `sessionSummaryContract` hub setting（#1771 移除）。
  - 沒有 `list_peers`；peer MCP 工具是 `inspect_peer`／`ping_peer`／`spawn_peer`。
  - Hub schema 的 V27 是 `steered`（#1771），V28 是 ACP cost（#1468，會清空 `usage_events`／`usage_scan_state` 再 rescan），V29 是 `session_read_states`（#2080）。從 .5（27）升上去是單向的，回滾要靠升級前的 `hapi.db` 備份。
- 同時 carry 多個改 composer 的個人 PR 時，要跑它們新增的 e2e。#1662 讓寬版 composer 直接顯示 model 按鈕，結果 #1836 的 luna-reserve e2e 在 1280px 找不到 `Settings`，unit test 完全沒抓到。
- `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <commit>^` 在這個環境可以非互動執行，用來把 `fixup!` 併進中段的 carry commit。完成後用 `git diff <舊 tree> HEAD --stat` 確認只動了預期的檔案。
- 結果：`release/v0.30.7.6` 以 force-with-lease 推到 `af29626a`，共 95 patches，`REPLAY_TREE_MATCHES=47dee910`、`REHEARSAL_OK`。完整測試結果：CLI 3,656、hub 1,642、web 3,846、shared 346、relay 118、desktop 5，全部通過。PR #30 的 body 用 `gh api -X PATCH` 更新。`main` 的 force-push 會被 Claude Code auto-mode classifier 擋下（Git Destructive），要使用者自己執行或加 permission rule。

### v0.30.7.6 發布與部署（2026-10-24）
- `main` 以 `--force-with-lease=main:6c3e23c4…` 推到 `af29626a82ba7538a23ac26726f141890272fd71`（PR #30 因 head=base 自動顯示 merged），tag `v0.30.7.6` 為 annotated。先前 auto-mode classifier 擋過這個 push；使用者明確核准後同一指令可執行。
- exact-main Test／iOS／android／fixtures／relay 與 tag Test 通過；Release Actions `40104169097` 成功，9 assets、8 payload checksums 相符。
- release branch 同 SHA 的 `windows-codex-mcp` 曾因 CLI test globalSetup `Hub did not become ready within 15000ms` 失敗（vitest 同時印 `No test files found`，那是 globalSetup 失敗的副作用，不是檔案遺失），exact-main 重跑通過，屬 Windows runner 偶發。
- 部署：oracle 本機裝 binary 後 `pm2 restart hapi --no-treekill --update-env` + `pm2 save`，13/13 session 存活；其餘交給 Mac `spawn-peer` session 依 skill 部署 mazu（hub/tunnel）、cthulhu、valkyrie、zeus、Mac、swop，全部 `0.30.7.6`，Athena 依政策排除。Hub 升級前備份 `hapi.db.pre-0.30.7.6-<ts>`（user_version 27），升級後 29、`quick_check=ok`。cthulhu／valkyrie 依例需 exact-PID handoff recovery；swop 自 handoff 後的 runner 脫離 Scheduled Task，需停掉該 exact PID 再 `Start-ScheduledTask`。
- Codex disposable smoke 不接受 `--permission-mode auto`（`bad_args`），codex 實際可用值為 `default/read-only/autoReview/yolo`（`safe-yolo` 能過 CLI 參數檢查，但 shared runtime 會拒絕，spawn 只回 502）；mazu 第一次 spawn 因 NFS 冷快取 `spawnSync codex ETIMEDOUT` 回 502，重試成功。
- Retention 清約 1.51 GiB；mazu／Mac 的 `0.30.7.5` 因 live session 保留，NFS 共用 `0.30.7.4` 與 `.nfs*` 因 Athena 無法檢查而保留。gist／skill Current Policy 與 `hapi-ctl.sh` 預設版本尚未更新到 `.6`。

## 2026-10-25：舊 PR 整理、#1771 拆解與下一版 carry 變動

- 使用者決定：上游關閉 #2024（#2069 已在 hub 端補 localId，涵蓋 peer 訊息）、#1836（維護者已 revert Luna Reserve fallback #1780）、#1542（內容已併入 #1597，只留 #1597）、#1771（約一萬行、213 檔、混了多件事）。#1771 的 prompt 移除使用者表示不是自己能決定的，先不處理；其中 OpenCode／Grok 的 Plan Mode 是靠注入 prompt 實作，移除 prompt 會連 Plan Mode 一起拿掉。
- #1771 改成兩個小 PR：#2054（#2053 修正，從 +98/−8 縮到 +25/−5：保存最新 keep-alive，connect 時一併送出；不需要「第一次 keep-alive 走非 volatile」的分支）與 #2167（`spawn_peer` + `ping_peer` `waitForReply`：caller 的 CLI 輪詢 peer，回合結束且連續兩次 idle 後，把回覆 POST 進 caller session 喚醒它；不改 hub）。理由：只靠 ping_peer／inspect_peer，caller 結束回合後就沒有東西會喚醒它，在 tool call 裡阻塞等待又會碰到 MCP timeout（Codex 預設 60 秒），所以就算 agent 夠聰明也很難互動。
- 上游已有的等價能力：`POST /api/machines/:id/spawn`、`/sessions/:id/{messages,abort,archive}`、`DELETE /sessions/:id`、`GET /machines`，CLI 的 `pingPeer.ts` 已有 JWT 交換與 REST client；session summary prompt 在上游預設就是關閉（設定要 `=== true` 才開）；shared Codex 已把 `thread/name/updated` 同步回 HAPI metadata。#1771 的 Codex 標題產生只是為了補 prompt 移除後少掉的 `change_title`，不是獨立 bug。
- 上游已有很多他人的 peer PR（#1922 spawn-peer remit +6.6k、#1618、#2021、#2047、#1825、#2152、#2071），維護者都沒有回應；新的 peer 功能盡量做小，避免和這些重疊。
- 下一版 fork：不再 carry #1771／#2024／#1542；carry #2054、#2167；疊上 fork 分支 `fork/session-control-skill`（stacked 在 #2167 上），它保留 #1771 的 skill 安裝器，並附上重寫後的 `hapi-session-control` skill（改用 spawn_peer／waitForReply）。#1836 上游已關閉，是否改用 overlay 保留，到發版時再請使用者決定。已 carry 的 #2052 `HAPI_DISABLE_PEER_TOOLS` 需要一併隱藏 `spawn_peer`。（2026-10-27 已在 #2167 分支完成：`PEER_TOOL_NAMES` 納入 `spawn_peer`，`spawnPeer()` 也會拒絕。）現行 skill 用到的 `stop-peer`、`--remit-id` 等參數會一起消失。
- 2026-10-27 #2167 merge upstream/main 後，經過 6 輪 HAPI Bot 審查，在 d3ab115a 拿到乾淨的 pass，CI 全綠，等上游 merge。教訓：只靠 `thinking`／連續 idle 次數來判斷 peer 回合是否結束，bot 會一直找到新漏洞（佇列中的任務、Claude 啟動慢、中途被打斷、後續回合）。可靠的訊號是任務的 `localId`：上游 #2069 讓 REST 送出的訊息在被消費前維持 uninvoked。任務不在最新一頁時，改查 `POST /sessions/:id/messages/queued-state`，查不到代表已被取消（hub 會刪除取消的訊息）。另外發現 `claudeRemote` 只在第一回合（system/init）設 thinking，後續 prompt 沒設；已在 #2167 一起修正（推入 prompt 時呼叫 `updateThinking(true)`）。
- 2026-10-28 #2167 重做回合結束判斷（使用者質疑「回覆可能遺失」）：改用 launcher 在回合結束且佇列為空時寫入的 `ready` 事件列（agent row `{type:'event',data:{type:'ready'}}`），條件是任務已消費、非 thinking、最後一筆非 user 列是 ready，並連續兩次輪詢成立；ready 被丟掉時有 10 分鐘 idle fallback。Pi 不送 ready，`spawn_peer` 直接拒絕。轉發改為全文（不再截成 1,200 字）。hub 的訊息排序鍵是 `invokedAt ?? createdAt`，所以以任務列為錨點往後翻頁；`afterSeq` 最小為 1，cursor 要用 `{at: at-1, seq: MAX_SAFE_INTEGER}`。同時消費的 prompt（相同 invokedAt）視為同一回合；hub 不記錄訊息是否為 steer，所以後面還有其他 prompt 時，兩段回覆都附上。hub 也沒有「以 peer 身份貼訊息」的功能（`sentFrom` 寫死 `webapp`）。上游 #2071 已加入 `ping-peer-` localId，取代我自己的版本。再經 5 輪 bot，在 7571265b 拿到乾淨的 pass，CI 全綠，等上游 merge。
- 陷阱：#1771 的 skill 安裝器掛在 `bootstrapSession`，CLI 單元測試會呼叫它，而測試 setup 只隔離了 `HAPI_HOME`、沒有隔離 `HOME`，所以會寫進真實的 `~/.claude/skills`、`~/.codex/skills`。以前寫入的內容和正式版相同，所以沒人發現；改寫 skill 後第一次跑測試就覆蓋了正在使用的 skill，已從 `~/.hapi/runtime/<version>/skills/` 還原。overlay 在 `cli/src/test/setup.ts` 把 `HOME` 指到測試用的暫存 home；`agentCliGuard.test.ts` 原本用字串 `toContain(join(tmpdir(),'hapi'))` 判斷，會誤中 `hapi-test-*` 前綴，要加上 `sep`。
- 平行跑 CLI 測試一定要各自使用不同的 `TMPDIR`：另一個 vitest 用同一個 TMPDIR 時，會刪掉 `hapi-test-config.json`，導致 257 個檔案 collection 失敗（`Missing isolated hub config`），那不是程式碼的問題。

## 2026-10-25：v0.30.7.7 發版（新 base 4233bf87）的已驗證陷阱

- 結果：`main`／tag `v0.30.7.7` = `702fea01c`，94 patches，`REPLAY_TREE_MATCHES=76102ffe`、`REHEARSAL_OK`。新 carry：#2165（側欄 re-render）、#2153、#2158、#2159；#1948 依使用者決定拿掉；#2164 drop（#1771 已刪掉它要修的 per-flavor display prompt）；#2097／#2098／#2101／#2102／#2042 已進官方 base。Release Actions 成功，9 assets／8 checksums 相符；exact-main Test／iOS／android／fixtures／relay 綠。
- 換新 base 時，最省力的重放方式：`git am -3` 依序套上一版的 patches，跳過已進 base 的那幾個。repo 開了 `rerere.enabled`，上一版解過的衝突大多自動解掉；寫個小腳本在「沒有 conflict marker、`git rerere remaining` 為空」時自動 `git add` + `am --continue`，其餘停下來人工處理。`.github/workflows/test.yml` 的 e2e spec 清單每個 carry 都會加一個，衝突時取兩邊的聯集。
- 上一版的 carry patch 若是疊在被拿掉的 PR 上解衝突（例如 #1662 疊在 #1948 上），改用 `git merge --squash refs/pr/N` 從 live head 重新產生，比修舊 patch 乾淨。
- 本 session 的環境有 `HAPI_DISABLE_PEER_TOOLS=0`（HAPI session 會注入），它是機器層級 override，會讓 #2056 的 5 個 peer-tools snapshot 測試失敗。跑測試要 `env -u HAPI_DISABLE_PEER_TOOLS`。`sync-from-upstream.sh` 的測試階段不會清掉它；tree 相同時用 `--skip-tests` 並引用已通過的完整測試。
- Manifest 規則（`UNRECORDED_FORK_COMMIT`）：上一版 `origin/main` 每個 commit 都要出現在某列的 source。carry 列的 source 是 PR head，不是舊 fork commit，所以舊的 carry commit 一律要另外列成 `drop`。regenerate 列的 source 填對應的舊 commit。用腳本依 subject／PR 編號對應產生 manifest。
- 發版過程中使用者的其他 session 會同時整理自己的 PR：merge main 進來（head 變了但 effective diff 不變 → 記為 refresh），或直接關閉（#2024、#1836、#1542；#1771 在 18:59 關閉時本版已過稽核）。rehearsal 前、以及 push `main` 前，都要重抓 live open PR 清單比對；被關的 carry 要從 queue 移除（`rebase -i` drop，確認移除量等於原 patch 的反向），被併入後繼 PR 的（#1542→#1597）保留 patch、改寫 subject 記成 overlay。
- 遺留給下一版：v0.30.7.7 仍含 #1771（關閉時間晚於稽核、早於發布，已在 fork PR #31 留言說明）；下一版依計畫改 carry #2054、#2167 和 `fork/session-control-skill`。#1836 是否用 overlay 保留，由使用者決定。
- Gemini 在 release PR 上的 critical finding「`MetadataSchema` 未 import」是誤報（第 39 行就有 import，typecheck 通過）；回覆說明後視為通過。
- 部署（2026-10-25）：oracle 本機裝好後 `pm2 restart hapi --no-treekill --update-env` + `pm2 save`，runner 是 `0.30.7.7`、`treekill=false`，舊 binary 上的 3 個 session 經過 idle-wait 後用 `stop-peer` 遷移；自己的 orchestrator session 留給使用者處理。oracle 刪掉沒人用的 runtime `0.30.7.5`（175 MB），`0.30.7.6` 因為自己的 session 還在用而保留。其他主機交給 Mac 的 `spawn-peer` session：mazu Hub（升級前有 DB 備份，schema 29）、mazu／cthulhu／valkyrie／zeus／Mac 的 runner 經 hub 回報都是 `0.30.7.7`，公開 Web 也已提供 `.7` 的 assets。swop 在部署期間 SSH 逾時，之後從 hub 機器清單消失，部署結果未確認。部署 session 還在做 retention 時，使用者轉而要它做效能檢查，所以其他主機的舊 session 遷移與 runtime 清理並未完成確認。
- 從 oracle 只能部署 oracle 本身：mazu 和 cthulhu 的公開 SSH 都會逾時，其他 alias 無法解析。其餘主機一律交給 Mac 的 session。另外，`hapi machines --json` 的結果不含版本，要查版本請用 hub `/api/machines`（先用 CLI token 換 JWT），讀 `metadata.happyCliVersion`。
