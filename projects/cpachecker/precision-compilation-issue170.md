---
title: CPAchecker #170 CFA-native precision compilation
scope: project
project: cpachecker
tags: [vguide, cegar, predicate-abstraction, precision-compilation, cfa, issue170]
status: active
created: 2026-09-03
updated: 2026-09-04
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

# 後續 pass：Issue #172（已完成）

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

更進一步核對第一輪 formal proof artifact：interpolant 0/1 已分別在 `N52`/`N57` 含有 native
bitvector `main::i@3 = 0`，後續 local precision 也確實保存 `i = 0`；缺的是 verifier 需要的
較弱 abstraction vocabulary `i >= 0`，不是數值事實本身。這使 #172 的最小突破口成為
**proof-guided equality order projection**：只對 authoritative assignment/interpolant equality
產生固定的兩個 order consequences，再用既有 frame rule 放置，而不是掃描常數或建立一般
consequence generator。`PRECISION_ONLY` 只是要求 CPA 追蹤該原子，不宣告它是 invariant，
所以首次 path 的 placement 不需要先證明它跨過 back-edge/`i++`。

CPAchecker 已有 `FormulaManagerView.splitNumeralEqualityIfPossible`，能把 numeral equality 變成
`<=`/`>=`；但 `extractAtoms(..., true)` 只保留 split list 的第一個元素再保留原 equality，且
測試明示方向受 solver 的 equality argument order 影響。全域
`cpa.predicate.refinement.splitItpAtoms=true` 因而不能作為 #172 的 byte-stable 雙向 projection
contract；compiler 應直接消費完整 split result，並以 C declaration/type 決定 signedness。
現有 helper 對 bitvector comparison 寫死 signed=true，也不能直接用於 unsigned contract。

`PathFormulaManager.buildWeakestPrecondition` 雖有公開 API，但預設
`handlePointerAliasing=true` 時 `PathFormulaManagerImpl` 不建立 `CtoWpConverter`，呼叫會直接丟
`UnsupportedOperationException`。因此 #172 不應把 WP 可用性當前提；較小且與目前 runtime
相容的 certificate 是用 `PathFormulaManager.makeAnd` 建 exact native SSA transition relation，
再驗證 `source_evidence && transition => target_atom`。WP/SP 只作為 semantics/certificate，
不負責無界合成；候選仍由 assignment/proof equality 的固定 projection 限定。

#172 的 C3b gate 維持 5/5：四個 v1 regression facts 不能退步，且 `nested9` 必須完整恢復
`i >= 0 @ {N52,N57,N62}`。

PR #175 已合併為 `25689ef90128695709a6c758db518ee46dda244f`。實作沒有建立完整
assignment SP/WP，而是只接受 aligned native interpolant 中、能與最近 direct scalar integer
assignment 的 post-state equality 雙向等價的 equality；再固定產生 native `<=` 與 `>=`，用
assignment LHS 的 C type 決定 signedness/width，並重用 v1 scalar frame transport。self-dependent
SSA collision、call、pointer/pointee、partial write、cross-function、ambiguous/missing equality 與
solver certificate failure 都 fail closed。所有輸出仍是 `PRECISION_ONLY`，dump schema 為
`cfa-precision-compiler-v2`。

Production indexing 的重要細節：`PredicateCPARefiner` 的 `abstractionStatesTrace` 不含 root，
且明確與 `BlockFormulas` 等長；去掉最後 error state 後，interpolant `i` 與
`abstractionStatesTrace[i]` 配對。不能套用一般「states 比 interpolants 多一個，所以 index +1」
的假設；既有 `VGuideAnalysisDumper` 也是 same-index mapping。正式 `nested9` dump 在 N52/N57 的
proof equality 也驗證了這個 alignment。

Targeted `CfaPrecisionCompilerTest` 14/14 通過；完整 ECJ、javac、JUnit、3,880 configuration
checks 與 Checkstyle 通過。SpotBugs 缺 Eclipse classes、Forbidden APIs 70 errors 都與相同 base
commit 完全一致，變更檔沒有新增 violation。PR review 的 Swear Review 因 OCR infrastructure
failure 無可用結果；Gemini 對 exact head `59f99e25bc` 的第二次 review 明確回報沒有 comments。

## C3b 正式結果（2026-09-04）

preregistration：`https://github.com/swear01/cpachecker/issues/172#issuecomment-5531094745`；
harvest：`https://github.com/swear01/cpachecker/issues/172#issuecomment-5531204306`。正式 merged
runtime 為 `25689ef90128695709a6c758db518ee46dda244f`，使用第一次即 idle-ready 的
`valkyrie`、P-core affinity `0,2,4,6,8,10,12,14` 與 HAPI attached job；不作 timing claim。

- `hh2012-ex1b`: N24，TRUE，5 refinements，pass；
- `nest-if3`: N47，UNKNOWN，15 refinements，pass；
- `nested-1`: N47，UNKNOWN，22 refinements，pass；
- `nested3-1`: N24，UNKNOWN，85 refinements，pass；
- `nested9`: `i >= 0 @ {N52,N57,N62}`，TRUE，2 refinements，pass。

因此 C3b **5/5 passed**；external model calls、wrong verdicts、integrity failures、scientific
failures 與 missing heads 全為 0。`nested9` refinement 1 已由 assignment edge occurrence 30、
proof-source occurrences 31/34 產生 signed 32-bit `PROOF_EQUALITY/GREATER_OR_EQUAL` origins，
equivalence/implication flags 皆為 true，並放到三個 required heads。這證實最小突破口是
proof-guided equality order projection，而非完整 assignment SP/WP。

完整 artifacts：
`<remote-home>/cpachecker-experiments/runs/issue172_equality_projection_c3b_20260904`。
summary SHA-256 為 `665185083aac13c6058e11437d4947147a0e46265b7a4005cf405a7b710932a6`，
48-file ledger SHA-256 為 `37bc297b0bd12c71434fd0b536051e89330412040d6841c6185cd1c3d697b6df`。
#172 已完成關閉，C3 stop condition 已清除；#170 C4 在研究機制上解鎖，但仍需另行
preregister，且既有 #174 native MathSAT stability blocker 沒有被本結果解決。

# C4 consumer gate：Issue #173（2026-09-04）

使用者決定把 `nested9` 的 assignment-derived 缺口留給獨立 #172，並讓本 session 在四個
guard-transport-compatible fixtures 上繼續。#173 依 #138 consumer gates 分離 generation、
validation、trajectory、verdict 與 cost；runtime 固定為
`6091a95c5ca12d548b2d2c9663c06aad5bfec1a0`，正式 replay 外部模型呼叫為零。

已完成且可接受的 fixture-level consumer evidence：

- `hh2012-ex1b` stock `UNKNOWN ×2`，compiler-only `TRUE ×2`；exact LLM-only 與
  combined replay 都是 `TRUE ×3`。compiler-only 在沒有模型的情況提供了足夠的 abstraction
  vocabulary，這是 first pass 的明確 consumer-positive anchor。
- `nest-if3`、`nested-1`、`nested3-1` 的 completed compiler-only cells 都維持
  `UNKNOWN`；candidate 確實注入並改變部分 trajectory，但不能把 refinement 差異當作 solve。
- live generation 24/24 slots，59/60 calls，7 `TRUE`、17 `UNKNOWN`、0 `FALSE`；59 calls
  共 94,634 prompt、38,997 completion、133,631 total tokens。完整 candidate set 保留，
  parser rejection 為零。

完整四-arm C4 **沒有完成，也沒有 aggregate result**。三個分別凍結的 formal roots 發生
native abort；最後一次是 `nested-1` combined draw-1，refinement 19 後 exit `-6`。保存的
`hs_err_pid3129964.log` 定位為 `libmathsat5j.so` 的 `Hashtable::find` /
`Mathsat5NativeApi.msat_make_term`，由 `FormulaManagerView.instantiate` 經
`PredicateAbstractionManager.getRelevantPredicates` 呼叫；native thread stack 尚有約 1013 KiB，
不是 Java stack exhaustion。依預註冊的零額外 native-failure rule，recovery budget 已耗盡，
不得再補 draw、把 crash 記作 UNKNOWN 或靜默 fallback。runtime blocker 另立 #174。

主要 immutable evidence 位於：
`<remote-home>/cpachecker-experiments/runs/issue173_guard_transport_precision_consumer_20260903/`。

- generation composite：`96c2fe5d02de7a31bde40a2aa290ac95fa0c95fca74938a49ebaa8e6fec88e5a`
- generation ledger：`81e9f33f6c4227fe037c4a628cb2ae708a311a9fe4521bc82e954dbdfad4c86e`
- attempt-2 stop ledger：`51018b75fa109d91f435613f85042059df5813977e4e160abcee39160a681b61`
- qualified nest-if3 combined ledger：`af47c3834ad0b9900aa1e6fa6b8e66e6480b1e9f5dadeb61787fdd7b23cc09ef`
- final stop ledger：`f86968bb84eaf9db20dfc9cd07ce13813efbcf825bd676a6eac22d71fd628682`
- preserved hs_err：`ad3722d10f9f8b6f66a6461ec2dcde17a7e97eb71bcdd2c350bea3c18742ffa8`

研究決策：此 evidence 支持 precision compilation 在 `hh2012-ex1b` 是實際 abstraction
consumer，不支持「first pass 可解大多數 proof failures」。C5 不從不完整 matrix 晉升；先由
#174 解決可重現的 native runtime stability，再凍結新的 consumer gate。#172 保持獨立，處理
assignment-derived coverage。#170 保持開啟。

# 下一個 agent 的入口

Source of truth 是 GitHub issue
`https://github.com/swear01/cpachecker/issues/170`，並需讀 #138、#150、#165、#166、#172、
#173、#174。實驗/Wiki 規則與 artifacts 仍以
`<remote-home>/cpachecker-experiments/` 的索引和 GitHub Wiki 為準。C0-C2 已完成並合併；
C3 第一次是 4/5，#172 C3b 已補足為 5/5；#173 的 partial C4 只有 `hh2012-ex1b` 可作
consumer-positive claim，完整 C4 依 stop rule 停止。C3b 解鎖下一個 C4 設計，但不得重啟
#173 formal matrix 或晉升 C5，直到 #174 有 reviewed fix 並另行 preregister。不能用 generation
success 替代 verifier utility，也不能從這些 fixtures 宣稱 population、timing 或 publication
result。
