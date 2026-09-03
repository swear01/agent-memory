---
title: CPAchecker #170 CFA-native precision compilation
scope: project
project: cpachecker
tags: [vguide, cegar, predicate-abstraction, precision-compilation, cfa, issue170]
status: active
created: 2026-09-03
updated: 2026-09-03
---

# 研究決策

Issue #170 將 VGuide 的新主線定義為 **CFA-native precision compiler**：不再把
predicate formula 與 loop-head placement 一起交給 LLM 猜，而是把 program semantics、
CFA、spurious `ARGPath`、current precision 與 proof artifacts 編譯成可驗證的
precision delta。三個最小語義輸出是：

```text
antecedent_formula
consequent_head
preserved_variables
```

研究領域候選名稱是 **precision compilation** 或 **proof-obligation compilation**。
核心差異：invariant synthesis 尋找對 reachable region 為真的公式；precision
compilation 尋找足以分離 abstraction failure 的 Boolean vocabulary 與放置位置。
後者的 predicate 不必是全域 invariant，但必須能 soundly lowering 到分析 consumer，
並帶有可稽核 provenance/certificate。

# 已確認的動機

- #150 證明某些 verifier-effective predicates 需要完整且精確的多 head placement；generic
  nested-loop cue 沒有提升 exact head completeness。
- #165 在 `hh2012-ex1b` 顯示 explicit CFA obligation 或 generic cue 都能穩定觸發需要的
  cross-loop placement，但只有單一 base case。
- #166 固定三案例的 generation 結果只有 `nest-if3` 通過；`nested-1` 與
  `nested3-1` 未通過。八個可執行 consumer arms 全部維持 `UNKNOWN`。因此繼續微調
  prompt 不是足夠的主線。

# 已確認的實作邊界

核對基準為 `origin/main@4a14d72cc15dd38263a2430f4064b352a9b2782c`：

- `ARGPath.getFullPath()` 已能取得精確有序的 CFA edges，無法填補 path hole 時回傳空表。
- `PredicateCPARefiner` 呼叫 VGuide bridge 時只傳 `allStatesTrace.asStatesList()`，所以目前
  bridge 遺失 authoritative edge path。
- `StructuredCounterexampleBuilder` 因而把 `branch_conditions`、`ssa_values`、
  `assignments` 標為 unavailable。
- `PredicateStaticRefiner` 已用
  `PathFormulaManager.makeAnd(makeEmptyPathFormula(), assume)` 把 `AssumeEdge` 轉成 native
  formula，但只做 global/function placement，不做 downstream exact-head placement。
- `EdgeDefUseData` 已提供 direct defs/uses、pointee defs/uses 與 partial-def flag。
- `LoopHeadPrecisionInjector` 已能把 native `BooleanFormula` 以 location-specific
  `PRECISION_ONLY` predicate 注入；不需要新增文字 parser 或 solver abstraction。

C0 contract 已凍結於
`https://github.com/swear01/cpachecker/issues/170#issuecomment-5524304792`。C1/C2 已由 PR
#171 合併為 `origin/main@6091a95c5ca12d548b2d2c9663c06aad5bfec1a0`：bridge 保留
authoritative `ARGPath`，compiler 以 `getFullPath()`、native taken `CAssumeEdge` formula、AST
declaration-backed `MemoryLocation` support 和單一 caching `EdgeDefUseData` extractor 實作
same-function scalar frame transport。輸出直接成為 exact-node `PRECISION_ONLY` native
predicate，production prompt 未改，compiler-only 模式不建立或呼叫外部模型。

已驗證的 implementation 細節：

- compiler 只處理 spurious counterexample；feasible counterexample 不做 compiler work；
- path hole、cross-function、call、pointer/pointee write、partial definition、unsupported/null
  declaration、formula-to-declaration mapping failure 都產生穩定 typed rejection；
- certificate 的 transport interval 是 half-open，並記錄 source predecessor/successor、target
  node、sorted accumulated direct may-defs 與 frame rule；
- canonical dump 先固定排序與 dedup，再對「不含自身 `sha256` 欄位」的 canonical payload
  取 SHA-256；相同輸入 regression test 為 byte-stable；
- `LoopHeadPrecisionInjector` 必須先收集 reached set 的 distinct precisions 再 union；逐一從
  root precision 反覆覆寫會遺失較早 precision component；
- stock native delta 必須在 compiler injection 前歸因，否則 compiler output 會被誤記為
  pre-existing proof artifact。

驗證：39 個 targeted tests 通過；full checks 有 4,336 unit tests 與 3,880 config checks 通過，
Checkstyle/SpotBugs 通過。Forbidden APIs 為 branch 70、相同 base 77，因此未新增 violation。

# Soundness 邊界

MVP 的 frame rule 是：

```text
FV(g) ∩ MayDef(tau) = ∅
```

輸出只代表 observed path 上從 taken guard 到 downstream head 的
`path-preserved precision candidate`，不能當成該 head 對所有路徑成立的 invariant。
它可以作為任意 abstraction dimension 走 `PRECISION_ONLY`，但沒有額外 entailment proof
時絕不能走 `ENTAILED` 或 interpolant strengthening。

MVP 對 cross-function segment、unknown call effect、pointer/pointee write、partial def、
unsupported declaration 與 unresolved path hole 一律 fail closed。這是第一個 compiler
pass，不是研究上限。

# 大膽但尚未驗證的假說

以下是 #170 的 falsifiable moonshot hypotheses，不是既有結果：

- H1：在 frozen nested-loop integer cohort，至少 70% 具有 marginal verifier utility 的
  location-specific predicates 可由 CFA/path/proof artifacts 機械重建或 transport。
- H2：以 CFA cutpoint、dominance、def-use 與 abstraction locations 計算 placement，可消除
  至少 80% formula-correct/head-wrong failures。
- H3：native IR 可使 JSON/parser/scope/signedness/`head_not_on_trace` interface loss 歸零，
  並使相同輸入產生相同 precision delta。
- H4：SSA substitution、WP/SP 與 solver-checked preservation 可跨修改 assignment 編譯
  affine、phase、congruence 與 selected-index relations。
- H5：同一 proof-obligation IR 可 lowering 到 PredicateCPA 與至少另一種實質不同 consumer。
- H6：deterministic passes 之後，LLM 的最佳角色縮減為 missing-template proposal、成本排序與
  residual semantics；compiler + residual learning 在 held-out consumer-positive tasks
  嚴格優於任一單獨元件。
- H7：反覆出現的成功生成或 rejection pattern 可固化成新 compiler pass，逐步縮小
  generative search space。

# 完整研究階梯

1. frame transport；
2. assignment-aware symbolic transport；
3. join/dominance-aware placement；
4. solver-checked semantic preservation；
5. recurrence/loop predicate compilation；
6. interpolant、unsat-core、block-formula 驅動的 precision minimization；
7. learned residual passes；
8. multi-backend lowering。

第一個 implementation 仍只做 conservative same-function scalar frame transport，以隔離機制；
其餘階梯逐層 preregister、驗證並晉升為 deterministic pass，不能一次混進 MVP。

# C3 正式 mechanism replay（2026-09-03）

正式 preregistration：
`https://github.com/swear01/cpachecker/issues/170#issuecomment-5525840763`；結果：
`https://github.com/swear01/cpachecker/issues/170#issuecomment-5526005796`。runtime 為
`270790e3847c6307daf11fd6821e29c77d32bde1`，使用 idle-ready `valkyrie`、CPU affinity
`0,2,4,6,8,10,12,14` 與 HAPI attached job。所有 frozen input/formal artifact hash 重算
成功，external model calls = 0、wrong verdicts = 0、integrity failures = 0。

- `hh2012-ex1b`: `i < 100 @ N24`，pass，TRUE；
- `nest-if3`: `k < n @ N47`，pass，UNKNOWN；
- `nested-1`: `i < n @ N47`，pass，UNKNOWN；
- `nested3-1`: native unsigned `x < 0x0fffffff @ N24`，pass，UNKNOWN；
- `nested9`: `i >= 0 @ {N52,N57,N62}` 全部未恢復，fail，UNKNOWN。

因此 frozen all-five C3 為 **4/5、整體 failed**；依 stop rule 沒有 preregister 或執行 C4。
四個成功案例都是 taken-branch-derived fact；`nested9` 是 assignment-derived fact。這是支持
把 assignment-aware SSA/WP/SP transport 設為下一個待凍結 pass 的 residual-gap 證據，不是
該 pass 必然成功的證據，也不能據此主張 consumer utility、timing、solve rate 或 population
結果。UNKNOWN 必須保留。

完整 artifacts：
`<remote-home>/cpachecker-experiments/runs/issue170_precision_compiler_c3_20260903`。
第一次 pool attempt 因本機 `load/` 輸出目錄不存在，在任何 CPAchecker process、model call 或
formal result 之前失敗；應在 formal launch 前建立 harness 的 output directories。該事件已在
`PRELAUNCH_ATTEMPT.md` 與 issue audit comment 保留，正式 case 沒有重跑。

# 後續 pass：Issue #172

`https://github.com/swear01/cpachecker/issues/172` 已凍結為下一個獨立工作項：從 exact scalar
assignment semantics 與既有 proof artifacts 編譯 deterministic native consequence atoms，再
重用 v1 frame transport 做 location-specific placement。

精確缺口不只是「讓既有 predicate 穿過 assignment」：`nested9` 的 authoritative path 上沒有
taken `AssumeEdge` 直接陳述 `i >= 0`；compiler 必須先從 `i = 0` 等 assignment/proof semantics
產生該 abstraction atom。#172 將這設為可證偽 hypothesis；若 bounded SSA/WP/SP consequence
compilation 仍無法恢復 target，必須停止並另行考慮 recurrence/loop-predicate pass，不能為了讓
fixture 通過而把 recurrence、dominance/join 或 learned templates 偷混進 implementation。

C3 formal dump 的第一輪 occurrence/certificate 再核對顯示，首次 `N52`、`N57`、`N62` 分別是
path occurrence 31、34、37；到這三點的 direct may-def 依序為 `{i}`、`{i,j}`、`{i,j,k}`。
因此 frozen target 首先檢驗的是從 `i = 0` 編譯 `i >= 0`，再沿既有 frame transport 到三個首次
head；恢復這個 target 本身不需先跨過外層 `i++`。#172 仍另以 targeted TDD 約束 supported
assignment transformation，不能把 initializer-only recovery 誤報成完整 assignment transport。

#172 的 C3b gate 維持 5/5：四個 v1 regression facts 不能退步，且 `nested9` 必須完整恢復
`i >= 0 @ {N52,N57,N62}`。C3b 未通過前，#170 C4 仍 blocked。

# 下一個 agent 的入口

Source of truth 是 GitHub issue
`https://github.com/swear01/cpachecker/issues/170`，並需讀 #138、#150、#165、#166。
實驗/Wiki 規則與 artifacts 仍以 `<remote-home>/cpachecker-experiments/` 的索引和 GitHub
Wiki 為準。C0-C2 已完成並合併，C3 已誠實失敗於 `nested9`，所以不得直接啟動 C4。下一步由
#172 先凍結並驗證 assignment-derived scalar consequence pass；完成 C3b mechanism gate 後，
才可重新判斷是否具備 C4 前提。不能用 generation/recovery 成功替代 verifier utility，也
不能從五個 mechanism fixtures 宣稱 population、timing 或 publication result。
