---
title: MathSAT native crash in predicate replays
scope: projects/cpachecker
status: active
updated: 2026-09-06
---

# Issue 174 — MathSAT native crash in predicate replays (root cause + fix)

**Status (2026-09-05):** DONE — PR #176 merged to main (`36290f3264`, commit
`73bfbd8212`). Swear Review passed (no feedback). Canonical `ant all-checks`
(Ubuntu packaged OpenJDK 21): fix tree and base tree **identical** — 202/202
test classes, 0 test errors, 0 JVM crashes, checkstyle/spotbugs green; both
fail only on the pre-existing 70 `forbiddenapis` errors in `VocabularyGuide`
(main's #171 code). Branch
`fix/issue-174-mathsat-term-crash` (worktree `<worktree-root>/cpachecker-5be9d29c/issue-174`).

## Root cause

- CPAchecker pins `javasmt-solver-mathsat` to **5.6.11** in `lib/ivy.xml`
  (commit `3e09dadf0c`, 2026-04-15; upstream "update_to_intermediate_javasmt_w_mathsat_5.6.15"
  branch was never merged because 5.6.15 needs glibc ≥ 2.38 ≈ Ubuntu 24.04).
  JavaSMT 6.0.0 itself bundles MathSAT **5.6.15** — the pin swaps the native `.so` only.
- **5.6.11 is the last non-reentrant MathSAT release.** 5.6.12 release notes:
  "All binary files are now built in reentrant mode by default."
- JavaSMT 6.0.0 `Mathsat5SolverContext` has `USE_SHARED_ENV = true`: every query
  prover env = `msat_create_shared_env(cfg, creatorEnv)`, destroyed on close,
  while the long-lived creator env keeps `declare_function`/`msat_make_term`.
  CPAchecker predicate analysis creates/destroys a prover env **per query** →
  heavy shared-env churn on the creator env in 5.6.11 (non-reentrant).
- Observed crash: SIGSEGV si_addr=0xe2 in `msat::hsh::Hashtable<Symbol const*>::find`
  via `msat_make_term` on the creator env; stack canary "stack smashing" is the
  *symptom* (1013/1024 KiB stack free → not Java stack exhaustion).
- Intermittent because the corruption depends on allocation/timing, not input.

## Fix (one hunk in lib/ivy.xml)

Remove the `<exclude module="javasmt-solver-mathsat"/>` and the explicit
5.6.11 dependency; let JavaSMT 6.0.0's own module resolve →
`javasmt-solver-mathsat 5.6.15` (sha256 `3cf15b572b…f59eb345`, glibc ≥ 2.38,
self-contained: libc/libgcc only; prints "reentrant" in the version banner).
Fleet hosts (mazu/cthulhu/valkyrie 26.04, athena 24.04) all satisfy glibc 2.38;
container images unaffected. No CPAchecker code, verdict, or resource changes.

## Verification (mazu, Ubuntu 26.04)

- C-level stress (`msat_stress.c`: declare+make_term on creator env, shared-env
  solve+destroy per cycle, cross-env term asserts) 30k cycles × 10 preds:
  **both 5.6.11 and 5.6.15 pass** → minimal churn is insufficient; needs the
  full CPAchecker workload (deeper formula complexity / more env churn).
- A/B (mazu) — **complete matrix, 0 native crashes on both variants**:
  - formal lane (P-cores, exact frozen params): nested-1 10/10 + 10/10,
    nest-if3 10/10 + 10/10, nested3-1 stock 10/10 + 10/10 — all exit 0,
    identical UNKNOWN verdicts.
  - supplement (S-cores, 5 runs/case/variant): all clean.
  - extended nested-1 (S-cores, 20/20 each): all clean.
  - extended2 nest-if3 (15/15 each, `live_record_attempt001/006` cache):
    exit 1 on BOTH variants = deterministic LLM replay miss (same prompt
    `caf587ff…`; the 5.6.11 runtime that made the recording also misses on
    mazu → host/timing-driven, not solver-driven). Excluded from crash tally.
  → 5.6.11 does not reproduce on mazu (~60 runs; low-frequency,
    allocator/timing-dependent); the A/B is the no-regression proof for the
    reentrant binary, and mechanism + frozen backtraces carry the attribution
    (stated honestly in PR #176 / issue comments).
- Canonical `ant all-checks` (Ubuntu packaged OpenJDK 21) on fix and base
  trees: **identical** — 202/202 test classes, 0 test errors, 0 JVM crashes,
  checkstyle/spotbugs green; both fail only on the pre-existing 70
  forbiddenapis errors in `VocabularyGuide` (main's #171 code).
- Fix worktree: `ant build-project` OK; stock nested3-1
  on the fixed tree → `Using predicate analysis with MathSAT5 version 5.6.15
  (38b25e6e004e) … reentrant`, exit 0.
- `ant tests` on mazu (Temurin 21.0.10, glibc 2.43, libstdc++ 16): 11
  SMT-solver test classes crash (junit fork=true → one JVM per class) with
  `std::__codecvt_utf8_*::do_unshift` SEGV at `msat_create_env` →
  `msat::TermManager::make_bv_number` (MathSAT, static libstdc++ inside the
  .so) and in Z3 `convertValue` (system libstdc++16). **Crash-affected class
  set is IDENTICAL on base 5.6.11 and fix 5.6.15** (verified by diff of the
  11 `Running …`→`Errors:1` classes), and Z3 — untouched by the change —
  crashes in both. So these are PRE-EXISTING mazu-environment test crashes
  (solver .so libstdc++ locale facets vs newer system glibc/libstdc++), NOT a
  5.6.15 regression. Includes `InductiveWeakeningManagerTest` (matches upstream
  MathSAT SIGSEGV history `1713a6c64f`). Distinct from the #174 intermittent
  `Hashtable::find`/`msat_make_term` crash; formal single-solver runs are
  unaffected (all A/B runs clean, both variants).
- NOTE: replaying #173 LLM caches on **main** (25689ef) hits
  `LLM response replay failed without live fallback` (exit 1) because
  `59f99e25bc` (#175) changed VGuide prompt content (CfaPrecisionCompiler) →
  prompt-hash mismatch. NOT a MathSAT problem; A/B avoids it by using frozen 6091a95.
- The codecvt test-JVM crashes are the known Temurin/Zulu/Microsoft/Corretto
  JDK 21 issue (JDK-8379560) already documented in repo `docs/notes.md`:
  Temurin 21.0.10 (our formal-run JDK) crashes in `libstdc++ codecvt` when a
  JNI C++ solver lib loads; **Ubuntu packaged OpenJDK 21 does not** — running
  `ant all-checks` with `JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64` gives a
  fully green test suite on both base and fix trees (verified 2026-09-05).
  So: formal runs use Temurin (protocol) and are single-solver (no test-JVM
  exposure), while local `ant` gates should use the packaged JDK.

## Pitfalls / gotchas

- The frozen runtime worktree `runtime-6091a95` had a **stale**
  `hs_err_pid3129964.log` (the original #173 crash) at repo top level;
  `cp -a` carried it into A/B runtimes and a script's `cp -a "$runtime"/hs_err*`
  copy step could attribute it to a clean run. Purged before A/B.
- `msat_to_smtlib2_term` (declared in JavaSMT 6.0.0 `Mathsat5NativeApi`) is
  absent from the 5.6.15 `.so` — but CPAchecker never calls it (no tests
  reference it; only java-smt's own NativeApiTest does). Verified no CPAchecker
  code path touches it.
- The 5.6.15 module's ivyxml publishes `solver-mathsat` (= x64 linux .so on
  resolve) — resolution of `runtime-without-gpl` pulls it automatically.

## Replay cache paths (frozen #173)

- nested-1 combined draw1: `runs/issue173_…/live_record_attempt001/003_nested-1_combined_draw1/cache`
  (namespace `issue173-main-nested-1-combined-draw1`)
- nest-if3 combined draw1: `…/006_nest-if3_combined_draw1/cache`
  (namespace `issue173-main-nest-if3-combined-draw1`)