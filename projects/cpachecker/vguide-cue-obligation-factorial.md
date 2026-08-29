---
title: VGuide generic cue 與 CFA obligation 的 factorization
scope: project
project: cpachecker
tags: [vguide, predicate-generation, cegar, experiment]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# VGuide generic cue 與 CFA obligation 的 factorization

## 已驗證結論

Issue #165 以 `hh2012-ex1b` 做 frozen 2×3 single-case gate，factor 是 generic
cross-loop cue（off/on）與 obligation（none/target/matched irrelevant）。Primary strict
obligation 是 standard SMT-LIB `i<100@N24`。

結果：

| generic cue | none | target | irrelevant |
|---|---:|---:|---:|
| off | 0/3 | 3/3 | 0/3 |
| on | 3/3 | 3/3 | 0/3 |

Frozen decision 是 `generic_cue_saturation_pattern`。此案例中 generic cue alone 與
source/CFA-grounded target obligation alone 都足以產生需要的 cross-loop placement；cue 已開時，
target obligation 沒有額外 hit。不要只因 ordinary/target tie 就推論 obligation 無效，也不要據此新增
proof engine 或 obligation architecture；先把 generic cue 與 obligation 做 factorization。

Frozen 規則選出的完整五-candidate response 經 production parser/consumer exact replay 兩次：
`TRUE`×2、scheduled refinements `[1,3]`、candidate rejection `[[],[]]`，兩次 refinement
trace byte-identical。這排除了 parser/consumer rejection 作為該 tie 的解釋。

## 推論邊界

這只是 consumer-positive base case 的 capability/mechanism evidence。它不證明 population-level
interaction，也不支持 timing、solve-rate、speedup、PAR-2 或 cost-efficiency claim。其他案例仍可能
需要 instance-specific obligation；必須另行 preregister 並測試。

## Provenance

- GitHub: cpachecker issue #165（completed）；#152 保持在 frozen stop
- runtime: `origin/main@be2d2fffa5e44d55361b8e745ba464feefd778a7`
- artifact: `issue165_cue_obligation_factorial_20260830/RESULTS.md`
- final result SHA-256:
  `f6872b8f79adf5665fef924b15eddccafcaff2db1fe3a1cce593ba954dc76657`
- selected response SHA-256:
  `a32c8bfc0b88cac8108b9dd26f387ced2b6aa88bbb2f30e908e651a98ebf62d0`
