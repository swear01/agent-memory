---
title: MathSAT native crash in predicate replays
scope: projects/cpachecker
status: active
updated: 2026-09-06
---

- **Frozen predicate replay crashes in `msat::hsh::Hashtable<Symbol const*>::find` through `msat_make_term`.** #174 used JavaSMT 6.0.0 with native MathSAT 5.6.11. MathSAT 5.6.12 introduced reentrant builds; JavaSMT creates and destroys shared prover environments while the creator persists. The old stack/canary symptom occurred with almost all stack free.
- **Check the loaded library before attributing the result to the fix.** PR #176 removed the Ivy 5.6.11 override so JavaSMT resolves native 5.6.15 (glibc ≥ 2.38). Read the actual version and `reentrant` banner. A/B froze #173 runtime and replayed over 100 invocations with identical verdicts and zero native crashes in both arms. Simple shared-env stress passed both versions too; the intermittent old failure was not reproduced on mazu. Keep the original backtrace and upstream reentrant boundary as distinct evidence.
- **Newer main gives `LLM response replay failed without live fallback`, or the test JVM dies in codecvt.** Use frozen `6091a95` for that exact historical replay; #175 changed prompt bytes. The Temurin 21.0.10 codecvt crash affected the same class set across both native MathSAT options and also Z3. Use Ubuntu-packaged OpenJDK 21 for local gates. Fix/base completed 202 unit classes and shared the pre-existing forbidden API failure. [Build and JDK details](build-verification.md)
- **A supposedly clean run contains an old crash file.** Check each run's own PID and timestamps; copied runtimes can carry stale `hs_err` reports. Fixing this dependency does not reopen stopped #173 slots. [#170 follow-up boundary](precision-compilation-issue170.md)

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/cpachecker/mathsat-crash-issue174.md)
