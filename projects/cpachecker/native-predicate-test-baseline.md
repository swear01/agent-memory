---
title: CPAchecker native predicate test baseline on Java 21
scope: project
status: active
updated: 2026-09-06
project: cpachecker
machine: valkyrie
tags: [cpachecker, mathsat, java21, baseline]
---

在 2026-09-06 的 issue #37 worktree 驗證中，Java 21 下 `ant tests` 的 11 個既有 predicate/SMT 相關 forked tests 在 native code crash：主要 frame 是 `libmathsat5j.so` 的 `std::__codecvt_utf8_base<char32_t>::do_unshift`，另有 `libstdc++.so.6` 的 UTF-8 codecvt frame。`ant configuration-checks` 也出現 `libstdc++.so.6` SIGSEGV。這些 failure 不涉及 VGuide lifecycle diff；同一 run 的 `LlmPredicateLifecycleTest` 與 VGuide suites 通過。後續若要恢復 full Ant gate，先查 MathSAT native library/JDK runtime compatibility，不要為此 baseline failure 改 VGuide 邏輯。
