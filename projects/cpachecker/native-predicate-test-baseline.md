---
title: CPAchecker native predicate test baseline on Java 21
scope: project
status: active
updated: 2026-09-07
project: cpachecker
machine: valkyrie
tags: [cpachecker, mathsat, java21, baseline]
---

在 2026-09-06 的 issue #37 worktree 驗證中，Java 21 下 `ant tests` 的 11 個既有 predicate/SMT 相關 forked tests 在 native code crash：主要 frame 是 `libmathsat5j.so` 的 `std::__codecvt_utf8_base<char32_t>::do_unshift`，另有 `libstdc++.so.6` 的 UTF-8 codecvt frame。`ant configuration-checks` 也出現 `libstdc++.so.6` SIGSEGV。這些 failure 不涉及 VGuide lifecycle diff；同一 run 的 `LlmPredicateLifecycleTest` 與 VGuide suites 通過。後續若要恢復 full Ant gate，先查 MathSAT native library/JDK runtime compatibility，不要為此 baseline failure 改 VGuide 邏輯。


後續 PR211 read-only artifact 對照：11 個 codecvt crash signature 與本 note 相符
（8 個 MathSAT、3 個 libstdc++），全部使用 Temurin21.0.10+7；另有一個 JIT
`LinearScan::compute_global_live_sets` crash，不能混為同一已知原因或同一命令。
XML inventory、hs_err file count、最新 focused JUnit stdout 是不同證據層，不能沿用
早期「7 crashes」或舊 11-test XML 代表後來 14-test focused run。

完整 gate 的已驗證 Ubuntu packaged OpenJDK 配置見 `build-verification.md`。
明確設定 JAVA/JAVA_HOME/PATH 時仍須核對實際 JVM；override 可以繞過 launcher 的
正確預設。此次只比對既有 artifacts，沒有重跑 native case，也不據 signature alone
宣稱最終 root cause 或此次完整 suite 已通過。
