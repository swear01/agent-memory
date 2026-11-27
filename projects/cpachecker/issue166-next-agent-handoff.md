---
title: CPAchecker issue 166 next-agent handoff
scope: projects/cpachecker
project: swear01/cpachecker
tags: [vguide, issue166, handoff, schema11, z3, classifier]
status: active
created: 2026-09-02
updated: 2026-09-03
---

- **看到 #166 classifier 標記 `trace_relative_precision_loss`。** 在既有 observed trace 的 refinement 3/N24，`i < 100` 的 `block ∧ ¬target` 是 UNSAT、`abstraction ∧ ¬target` 是 SAT。refinement 4 兩者皆 UNSAT 為 retention control；synthetic `i == 100` 的 block 是 SAT。這是 model-free Z3 讀 accepted artifacts 的結果，未啟動 CPAchecker 或生成新回答。
- **想把 trace result 當 all-path invariant 或泛化。** 另凍結研究問題：全路徑 claim 要檢查 concrete entry／loop transition 的 initiation 與 inductiveness；replication 則要先凍結 provenance-based held-out cohort。不從 outcome 挑 case，分開量 generation、transport、abstraction precision 和 consumer usefulness。此結果也未隔離 candidate 5 的因果效果。
- **重用舊 replay 發現 request hash 不符。** 本案 retired `DEEPSEEK_MODEL` 被 Java 忽略，實際 fallback 走另一模型；另有 prompt/context 字節差異和不同 arm 的後續 request sequence。保留原錯誤與 exact-hash 條件，不猜 hash、改 production prompt 或重新執行已關閉的歷史 packet。看 [實驗 provenance](vguided-experiment-provenance.md) 和 [Python／Z3 環境](issue166-python-environment.md)。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/cpachecker/issue166-next-agent-handoff.md)
