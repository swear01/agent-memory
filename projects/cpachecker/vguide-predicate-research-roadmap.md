---
title: CPAchecker VGuide predicate 研究主線
project: cpachecker
tags: [vguide, predicates, cegar, nested-loops, research]
status: active
created: 2026-08-26
updated: 2026-08-26
---

# 核心目標

先證明至少一組生成 predicate 能在原本 UNKNOWN 的 case 中，經既有
PredicateCPA/CEGAR lifecycle 造成可重現的 trajectory 或 verdict 改變；再用同一機制
挑選 held-out case。不要從單一 predicate 的表面 novelty、初始 precision、solve count
或 timing 直接推論整體有效。

# 已確認的 base case 與 generation gap

- `c/loop-lit/hh2012-ex1b.yml` 的完整 delayed oracle policy 在 matched replay 中把
  UNKNOWN 改為 TRUE（Issue #147/#148/#149）；這先證明 consumer 能使用跨 loop-head
  predicate，不代表 LLM 已會生成。
- #148 的 blind test 中，模型在 3/3 reasoning traces 都注意到 enclosing outer guard，
  但在 0/3 final candidate sets 把等價 predicate 放到 inner head。問題是 candidate
  selection：舊 prompt 要求 concrete-state-pair separation，會刪掉對 concrete states
  冗餘、但對 location-specific abstraction 有用的 predicate。
- PR #155（merge `f478fdb76b907c067759f9518661b39c8ae79597`）把 production cue 改為：
  `Add a candidate only if it separates proof-relevant concrete or spurious abstract states at that head; for nested loops, consider inherited outer-guard facts over variables unchanged there.`
  這是 answer-free direction cue。#149 的 Muse frozen blind A/B 已完成：control
  final-JSON strict hit `0/3`，treatment `2/3`；最低編號 treatment hit 的完整五項
  response 經 production consumer replay 為 `TRUE/TRUE`，兩次皆 exact request/list、
  zero rejection。這證明該 cue 在 HH2012 base case 有 generation 與 consumer utility，
  但 `2/3` 也顯示輸出不是 deterministic，不能外推 population hit rate。

# 後續工作佇列

- #149：已達 acceptance 並關閉；implementation 是先前已 merge 的 PR #155，本次完成的是
  post-merge frozen experiment evidence，沒有新的 CPAchecker production commit/PR。
- inherited outer-loop cue 的下一步：另開 follow-up，選 2–3 個 held-out、consumer-positive
  nested-loop cases，用相同 control/treatment、strict final-JSON scoring 與 complete-response
  replay 檢查 generalization；不得把 #149 的單題 `2/3` 當作一般成功率。
- #150：location-complete placement；先用 remove-head 證明每個 head 都是 consumer-positive。
- #151：從 coupled updates 生成 affine conservation relation；先過 G2 consumer gate。
- #152：提供 deterministic/source-grounded inductiveness obligations；加 irrelevant-obligation control。
- #153：用 bounded refinement feedback 做 incremental lemma completion；和等 call/token budget 的
  one-shot 對照。
- #154：已由 PR #156 修復並保留 strict ECJ gate；build/fleet 驗證分層見
  `build-verification.md`。

# 實驗邊界

每條研究線都先凍結 consumer-positive base case、gold-free prompt、scoring 與 stop rule；
generation、validation、trajectory、verdict、cost 分開記錄。完整生成 response 必須原樣 replay，
不能只挑成功 atom。正式 model call 必須重用 Java `PredicateProposalClient` transport；參見
`vguide-experiment-transport.md`。研究規格以 GitHub Issues/Wiki 為準，artifact 在
`<remote-home>/cpachecker-experiments/`。
