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
- #158 held-out gate：source-matched census 只有 `nested_5`、`nested_6` 兩個非 HH、Stock-UNKNOWN
  nested-loop cases，沒有用 Stock-TRUE 或 #150 的 `nested9` 補成三題。兩題的 matched-empty
  都在 300 CPU-s 維持 `UNKNOWN`，但從 empty trajectory 凍結的 complete ancestor-guard
  schedule 在注入前兩層後改變了後續 spurious trace；`nested_5` 的 `N28/N33` 共有 7 個
  `head_not_on_trace` rejection，`nested_6` 的 `N29/N34/N39` 共有 12 個。因此兩題都在
  deterministic consumer-positive gate 停止，沒有做 live blind A/B。這不能判定 cue 在
  held-out generation 成功或失敗；只證明不能把 empty-arm 的未受干預 trajectory 當成
  intervention arm 的靜態 location schedule。若續做，必須另開並預註冊能依當下 trace
  決定合法 head 的 reference gate，不能在 #158 內手改 head 或加 draw。
- #162 semantic-capacity follow-up：在最新 main `4da075b682` 直接把完整 proper-ancestor
  `variable<6` family 以 location-specific initial precision 載入，排除 trajectory scheduling
  干擾。`nested_5` 精確載入 10 個 local predicates、`nested_6` 載入 15 個；兩者 exit 0
  跑到 300 CPU-s 仍為 `UNKNOWN`。因此這兩題對「只有 inherited guards」是
  consumer-capacity-negative，依 prereg 在 adaptive replay 與 live Muse 前停止（external
  responses 0）。後續若要測 cue 泛化，須先找另一個 source-matched consumer-positive case，
  或另開並預註冊包含 exit equalities 等不同 predicate theory；不能回填 #162。
- #150：location-complete placement；先用 remove-head 證明每個 head 都是 consumer-positive。
- #151：從 coupled updates 生成 affine conservation relation；先過 G2 consumer gate。
- #152：提供 deterministic/source-grounded inductiveness obligations；加 irrelevant-obligation control。
- #153：已完成並關閉。Frozen complete consumer group 兩次為 `TRUE`，strict subset 與
  zero-call lifecycle 為 `UNKNOWN`。在相同 4-call / 4096 completion-token ceiling 下，兩次
  incremental natural replay 都是 `TRUE`（106 refinements、10 個 solver-distinct bindings），
  matched one-shot 分別是 `UNKNOWN`（129/135 refinements、0 bindings）。Production validation、
  MathSAT 跨 round dedup 與完整 sequence replay 均通過，錯誤 verdict 為 0。這只支持 later-round
  feedback 產生替代的十個 `j` bindings；模型仍只生成 frozen group 的 1/3，不能宣稱 literal
  group completion、population hit rate、timing 或一般模型品質。
- #154：已由 PR #156 修復並保留 strict ECJ gate；build/fleet 驗證分層見
  `build-verification.md`。

# 實驗邊界

每條研究線都先凍結 consumer-positive base case、gold-free prompt、scoring 與 stop rule；
generation、validation、trajectory、verdict、cost 分開記錄。完整生成 response 必須原樣 replay，
不能只挑成功 atom。正式 model call 必須重用 Java `PredicateProposalClient` transport；參見
`vguide-experiment-transport.md`。研究規格以 GitHub Issues/Wiki 為準，artifact 在
`<remote-home>/cpachecker-experiments/`。
