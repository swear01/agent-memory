---
title: CPAchecker native predicate test baseline on Java 21
scope: project
status: active
updated: 2026-09-08
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

## Stock benchmark crash is a separate qualification failure

2026-09-08 的 issue208 Stock stage24 在 Ubuntu OpenJDK21.0.12+8、MathSAT5
5.6.15 下，copysome2-1 出現 SIGSEGV：`libmathsat5j.so` 的
`msat::HashMultiSet<...>::begin()+0x8`，capture 記 exit -6 / signal6。
這是關閉 VGuide、零 provider calls 的 Stock，不能歸因於 LM predicates，也不能
當成前述 Temurin codecvt signature 已知原因。Ubuntu JDK 讓 build/focused tests
通過不等於 native benchmark 已全面穩定。既有 artifact 診斷由 issue215 追蹤。

三個 launcher exit0、24份紀錄的完整性與 hash 驗證通過，仍不表示實驗 gate
通過：task-level native crash 依預先停止規則阻止 Augmented/remaining194 admission。
保留 crash outcome 與原始218/24分母；不可刪題後把同次 checkpoint 稱為通過。
證據見 sibling experiments 的 `reports/issue208-final-runtime-20260907/`。

Issue215 的一次同機、同命令、同 native/JDK/source 的單題重跑（改為單題，
原 run 是 parallel8）在600.330CPU秒後為 UNKNOWN/timeout，
沒有 native crash；不可据此宣稱修復、歸因負載，或回填原 stage24。最近 issue180
同題也曾在 MathSAT5 5.6.15 下 timeout；root 核對 preserved library bytes，
其 Linux `libmathsat5j.so` hash 與新 run 相同（`3cf15b57…eb345`）。
因此舊5.6.11資料並非完整近期歷史，solver binary 升版不是這組觀察的已知差異；
JDK、整體 runtime/source、主機與 trajectory 的差異仍未隔離。
