---
title: CPAchecker VGuide predicate 研究主線
project: cpachecker
scope: project
tags: [vguide, predicates, cegar, nested-loops, research]
status: active
created: 2026-08-26
updated: 2026-09-26
---

# CPAchecker 研究主線與 LLM 使用授權

## 持續 LLM 使用授權 — 2026-09-23

使用者明確表示研究中可以直接使用 LLM，數十萬甚至百萬 tokens 的用量可接受。**CPAchecker 研究範圍內，依解題與驗證需要主動呼叫 LLM，不再因呼叫次數、token 數或一般模型費用逐批請示。** 這是使用者的成本容忍與持續授權，不是所有 provider 免費的價格聲明；百萬 tokens 也不是新設的硬上限。

- 呼叫規模依研究問題決定；可做足夠的候選生成、失敗分析與重複對照，不為省 token 只做幾次就停止，也不沿用 #278 的 3 次作為後續全域額度。
- 新實驗先記錄假說、題組、比較方式與停止條件，再直接執行。這是可重現性紀錄，不是新的人工批准關卡。探索和正式比較分開標示；已凍結的比較不事後補抽或換題，下一輪用新紀錄承接。
- 保留完整請求／回答、模型設定、actual usage、失敗與驗證結果；未知 usage 不當成零。判定仍看驗證器實際使用及正確解題效果，不能以候選數或 token 用量代替研究成果。
- 舊文件的「未授權新付費呼叫」、#269 的 192 次與 #278 的 3 次限制僅屬各自歷史 packet，不能再當成未來研究的授權障礙。#269 已停止 slots、#197/#215 correctness/native holds、Reserved44 及凍結原始證據仍保留。

來源：2026-09-23 使用者明確修正「幾十萬 甚至百萬 token 幾乎都是沒有成本 可以直接用」，並要求同步記憶、文件、QMD、issue。研究主索引為 #182／Wiki Research-Convergence。此授權針對研究用 LLM 呼叫；Codex/HAPI session 的 reasoning effort／Fast 偏好另見 [成本偏好](../../global/codex-cost-preference.md)。

#278 的單例必要關係 Goal 已完成：3 份原始 LLM 回答直接消費為 TRUE／UNKNOWN／TRUE，固定第一份完整回答兩次 TRUE、移除三個索引關係兩次 timeout，62/62 actual precision members。詳細限制見 [native C 邊界](native-c-predicate-boundary.md)。跨題可重複收益仍由 #182 追蹤。

## 三條路線的持續目標與完整集合驗收（2026-09-26）

使用者明確糾正：三條路線各設 goal，不能只完成第一輪或單題展示。有改善就擴大到完整 set；沒改善要分辨實作／表示／consumer／模型生成／證明與資源限制，嘗試合理修正並實測，不能把一次 timeout、截斷或 parser rejection 當作方向無用。只有可查核的原因、必要修正／有界負面結論與完整集合效果量具備才完成；不因 PR 服務或單 lane 問題停止其他可進研究。

共同主分母固定既有 hard218（224 父集扣原 6 infrastructure censor），另完整 array-cav19 13 題、array-examples/sorting*.yml 6 題及完整 sorting 命名 inventory、reducercommutativity 50 YAML（官方 unreach-call 28；其餘 22 是 def-behavior property，明列不適用）。不能挑成功題、刪失敗分母或把 family 成功率當成全218；這是已曝光的研究集合，不宣稱未見題泛化。Reserved44 不動。

#281 分開 Stock、保留 strict-bound safety 修正的 pre-global array、patched array；#282 分開人工 scaffold、LLM 自動發現與輔助 backend／CPA consumer；#287 INTEGER 成功必有每題原 C 接合，signed overflow gate 與 cast UF 並不足以涵蓋任意 unsigned/mixed comparison/pointer 操作。全部記 new/lost/wrong/unknown、coverage、整體 CPU 和模型成本；預先固定 fallback 與抽樣，事後最好結果 union 不冒充等成本 portfolio。正式 population 按既有 idle-ready／P-core protocol，候選整體 gate+verification 亦納入600 CPU秒預算。新完整集授權是 fresh matched research，沒有重啟舊#269 slot／#197/#215診斷replay包或宣稱舊缺陷已修。

主協調證據在 `cpachecker-experiments/reports/three-route-fullset-20260926/`，各路線為 `issue281-fullset-20260926/`、`issue282-fullset-20260926/`、`issue287-fullset-20260926/`；#280 與 Wiki Three-Route-Fullset 維護目前驗收。三位 Astra owner 各自有獨立 active goal；前輪結果是起點，不是這次 goal 的完成證據。

### 已驗證的 consumer 與算術限制

- **cast UF 不足以保留 unsigned 語義。** `ignoreExtractExtend=false` 的 INTEGER 路徑，在同寬 signed/unsigned 比較與 unsigned wrap 兩個原 C 反例給錯誤 TRUE；精確 BV 為 FALSE。既有 `cpa.predicate.encodeOverflowsWithUFs=true` 使這兩個控制恢復 FALSE，應先測現成功能，不必重寫 resolver。但這只是兩個控制，不能推論通用 soundness；no-overflow property 也不能代替明確 cast、unsigned 或 pointer 操作的接合審查。原 signed fragment 的排除條件仍有效。root 已逐份核對六個 raw logs／source hashes，見 `three-route-fullset-20260926/issue287-signedness-independent-audit.json`。
- **排序 importer 與模型失敗要分開。** Clang AST 同時列出 implicit builtin 與原 `abort` 宣告，機械產生兩份 ACSL contract 會先導致 parser error；排除 `isImplicit` function declarations 後，同一批原模型回答才進入真正 VC 檢查。安全 selection 的模型原先在 swap 後仍引用 `a[s]`，一次原始失敗 VC feedback 後自行改用當下 `a[i]`；完整原 C、未人工修改的22項模型候選經既有 JSON schema／註解 consumer 得到127/127。root 重建輸出逐 byte 相同，核對44個 invariant initiation/preservation 和原 assertion caller obligations。這是已曝光研究題的 auxiliary backend 證據，尚非 CPA 或完整集合效益；見 `three-route-fullset-20260926/issue282-selection-independent-audit.json`。
- **整體成本不能只看最後 checker。** gate、INTEGER 和必要 exact-BV validation 均計入同一600 CPU秒；各 phase 的 `prlimit` 是單 process 限制，還須核對 `RUSAGE_CHILDREN` 累計與正式 cgroup 總 CPU，再採納 verdict。證明義務全過和 consumer 實際使用只證明該例可用，新增解題仍需配對 baseline 與完整分母。

## 三條路線實際執行（2026-09-26）

使用者授權開始 #281 並允許 GPT-6 Astra 平行研究，已同時執行 #282／#287。各路線必須分開歸因，不能合計成「LLM 多解三題」。詳細證據為 `cpachecker-experiments/reports/issue281-execution-20260926/`、`issue282-execution-20260926/`、`issue287-execution-20260926/`；Wiki Breakthrough-Execution 統一索引。

- **#281 原題表示入口。** 同 base e76bd0dc 的原始 pair-symmetr2 是3 arrays／0 loops／UNCHANGED；安全辨識不變global後3 loops／PRECISE／0 retained，現成index對齊已足夠。v2原題N100000/ILP32/原assert範圍在60CPU arm TRUE；600CPU配對兩次B0都OS CPU limit而無reported verdict，B1兩次TRUE。此題無合法loop head、Stock+array已解，LLM arm不適用，0研究模型呼叫。mutable-global保持UNCHANGED/FALSE，zero/equal controls FALSE。
- **發現並修復的健全性缺陷。** v1 `N=INT_MIN` 的零次迴圈因共用ConstantComparison用C int做N-1而wrap，反例B0 FALSE／B1錯TRUE。v1停止且中止run不當timeout。v2以BigInteger規範化strict endpoint，要求index值域可在calculation type表示、rhs可表示、endpoint可在index type表示；不能把-2147483649做成int literal。INT_MIN/<與INT_MAX/>反例在v2 B0/B1全FALSE；PR288，11tests與Checkstyle通過。canonical all-checks與未改e76bd0dc同14個strictECJ錯誤，研究compile.warn建置不等於全綠。
- **#282 auxiliary原題proof。** sorting_bubblesort_ground-1.i，N100000/ILP32，只加ACSL註解且C tokens不變。Frama-C31/Alt-Ergo2.5.4 reference84/84兩次。原y assertion loop的`a[x]<=a[y-1]`直接做distance induction，無需另造all-pairs lemma/ghost loop。模型只在人工作好的prefix/frame/contracts scaffold中補y關係，原樣候選也84/84兩次；移除prefix或distance各剩一個未證義務，錯誤strict assertion未被證明。CPA仍UNKNOWN，不能算CPA/完整模型策略新增解題。
- **#287 既有整數表示。** 原輸入是reducercommutativity/avg10-2.i，N10；array abstraction因call/alias eligibility為UNCHANGED，不是已轉換後丟失sum。INTEGER BMC保留cast UF原題TRUE並重現，原C BV no-overflow、avg body/return-range與局部sum義務接合；精確BV MathSAT/Z3、CBMC仍timeout。rotate正確摘要是`Sum(x)+temp-x[i]=S`，中途Sum=S負控制FALSE。三次avg的loop counter累計33，k20不足，k64的bounding assertions保留。這是此題算術接合，不是通用INTEGER soundness宣稱；不需新增aggregate engine，沒有LLM增益。

研究Meta transport合計7 live calls，已知16908 tokens另3次usage unknown；不含Astra協作本身用量，也不是總token/美元費用。#282兩份整體方案截斷排除，只1份完整局部repair；#287兩份完整研究回答、1份截斷及1次原VGuide回答，VGuide12個local predicates注入後仍UNKNOWN。原始输出／失败／usage不丟棄。現有格式與resolver沒有重寫，廣泛跨題收益仍未完成。

## 優先路線與詳細計畫（2026-09-26）

使用者要求不同路線分開追蹤：#280 為總覽，#281 共同索引／陣列關係、#282 排序分解、#283 策略搜尋、#284 經驗證轉換、#285 輔助後端、#286 整體歸納、#287 reducer 總和守恆。之後授權選最有希望的一條做詳細計畫，選 #281；完整設計在 Wiki Shared-Index-Plan 與 `cpachecker-experiments/reports/issue281-shared-index-plan-20260925/PLAN.md`。此段記錄執行前規劃；後續實測見上節。

重新核對到重要的既有證據：9/22 `predicate-representation-goal-20260922/array-route.json` 的原始 pair-symmetr2 是 3 eligible arrays／0 loops／UNCHANGED；同批 N=100000 字面常數診斷版為 3 arrays／3 loops／PRECISE／0 remaining loops，匯出 C 已有 a/b/c index 對齊。該 probe 屬 runtime-4a6d353068，不能冒充目前 main 的結果或原題安全 verdict。目前 main e76bd0dc 的 TransformableLoop 仍明確排除 global bound。最短下一步是同版本重查，再安全辨識不變 global／沿用現成對齊；不能直接刪 global guard 或預設須實作共享索引。

若轉換後 Stock 已可解，記為表示改善，不能歸因 LLM；若已無 loop head，現有 loop-head schema 沒有合法注入位置，不是模型生成失敗。只有尚有證明需求與合法位置時進入 reference／20 份 LLM 初始批次及必要關係對照；20 不是新的全域用量上限。先排除原題 eligibility／表示問題，#283 的策略比較保持獨立。

## 開放突破研究：方案先於 predicates（2026-09-23）

使用者要求跨越目前表示法思考。研究建議（尚未證實解題增益）：LLM 先提出證明分解、需保留的關係與落地路徑，再產生 precision predicates 或待證 lemmas。首批深度案例為 pair-symmetr2 的同索引 a/b/c tuple／跨階段關係，以及 sorting 的 guarded last-pass＋adjacent-to-all-pairs lemma；checked fusion／tiling、auxiliary deductive/CHC、full-program induction 是替代路線。原表示／新表示 × 直接生成／證明方案與局部失敗修復的 2×2 才能分開歸因。不是開發通用 IR 的承諾，也沒有新原題 verdict。

已核對的關鍵界限：pair-symmetr2 的 N 是 global，local-bound 修補未覆蓋；現成 abstraction 是否遺失跨陣列關係仍為假說，不可直接判定。排序安全性只需到達終點時有序，不必額外證 termination／permutation；inner-loop 的前綴必須帶 swapped==0 條件，[1,2,0] 是無條件前綴的反例。小型 SMT／有限枚舉僅核對局部推理，未證明整個 C 轉換。62/62 是七次 verifier runs 的 precision membership 合計，第一份成功回答只有10候選，不能寫成62個predicates的解法。

資料：`cpachecker-experiments/reports/representation-breakthrough-20260923/REPORT.md`、`check_ideas.py`；Wiki Representation-Breakthrough／issue #182。精度提示不必先是 invariant；要作 assumption 的 lemma 則必須證明有效及足夠。獨立 solver 檢查 LLM 自寫的 SMT，仍不能證明該 SMT 忠實編碼原始 C；需要可信 frontend／可檢查的轉換義務。

以下為日期化決策與結果；當時的付費授權限制已由上方持續授權取代，不形成新的執行佇列。

# 歷史固定小組端到端決策（2026-09-21）

使用者明確接受收斂：不再以更多diagnostics、injected predicates、單题解释或準備完成
代替可重複新增正確解題。#182是唯一管理入口；#269比較同一runtime的Stock、一次生成、
分輪refinement回饋，使用相同總模型/驗證預算。已固定12題development面板（2正例加
10個既有hard218失敗；eligible116題，以完整hash排序選其他8題族），不事後換題、不動Reserved44。
已知正例或多次replay不算新的hard218收益，repair計入模型總額度。

PR #270 已修正首輪K被忽略、ensemble union過早截斷與失敗不扣round；
兩臂可關閉repair，詳見 [budget accounting](vguide-proposal-budget-accounting.md)。
固定兩次重複/三臂共72 verifier slots，最多192 HTTP（每次最多1024 completion tokens，
prompt tokens另計）；2026-09-21使用者明確同意此192次上限，root已對固定packet放行。
這不是後續研究的無限付費授權；benchmark結果尚未完成，沒有新收益可宣稱。

#259/#103僅支援能改變下一步決策的代表性生成/資訊/表示/reference診斷；
不要求每一題先有完整proof-adequate oracle才准整批進行。reference缺乏就標unknown。
固定pilot沒有收益時，交付負結果/限制並停止該輪；不能自動追加draw、換題或instrumentation。
至少兩個不同題族的開發失敗有預先規劃的重複淨增益且無未解釋新wrong後，才考慮擴大。

#139的深層Problem14歸因、#151已飽和affine cue、#173compiler矩陣與#101歷史差異暫緩。
#170大型compiler/IR、多agent/model sweep、舊224/Flash/DeepSeek與全維度ablation計畫
明確取消或整併，不宣稱那些假說已驗收成功。既有程式和原始證據不刪。
#54/#56的11個wrong、#197/#215 native/interpolation和#201工具問題保持獨立追蹤，
原hold不變；不把全部修好作為無關小型pilot的前置條件。

最初issue整理本身沒有付費/solver授權；後續僅#269這一輪取得上述明確有限額度。
平台舊goal仍usageLimited且拒絕同thread覆寫，不得假標完成。使用者已要求新thread接手；
2026-09-21已建立獨立HAPI/Codex root，並在該thread成功建立新的active Goal。
新Goal只負責既有#269固定pilot的完整核帳、剩餘範圍判定與go/pivot/stop決策；
沒有自動追加draw、換題或擴大full218的權限。實際thread/session與tool回執保留在
實驗report的successor-goal-receipt.json及successor-acceptance.json，不能用任務標題代替Goal回執。
操作目標仍由Wiki Research-Convergence/DR-022及#182控制。

交接時自動彙整為INCOMPLETE：56/72 terminal-qualified、143 observed HTTP starts，
Athena SSH exit255且boot identity已改變；兩個中斷slot與14個未啟動slot均保留。
這不是完整負結果；不得將unknown usage視為零，或覆寫中斷slot後宣稱沒有重跑。
接手先按新boot核對程序、slot與已用HTTP，再決定既有額度內尚可執行的明確範圍。
Cthulhu的唯一未啟動續跑已結束，不得再次啟動原host script或continuation。
舊目標與下面的日期化研究內容是歷史，不自動形成新的執行佇列。

## #269 successor final disposition（2026-09-21）

Successor在不重跑任何terminal/interrupted slot、不換題、不增加panel的前提下，使用原本
192 HTTP授權內真正未啟動的slot；Athena連續三次reboot/transport interruption後，所有72
slot均已被核帳：62個terminal-qualified、10個raw interrupted、0個unlaunched。中斷的
`run_meta.json`、CPA log、cache/dump與缺少terminal evidence均保留；不得把它們當成timeout
或negative solver outcome，也不得再重跑。

最終observed HTTP starts為167/192（包含中斷request）。原 collector 的147個usage
response可觀測、1個usage未知及2,123,623 tokens只涵蓋terminal slots，並非全部成本。
後續逐筆核對全部72 slots的HTTP start/end、request/content hash與raw response，
恢復中斷slots的19個回答（18有usage、1unknown）。全packet為167個completed HTTP／
165 observed usage加2 unknown；observed-only tokens為prompt 2,224,506、completion
90,313、total 2,314,819。unknown為t06-r2-one_shot call4與t10-r2-feedback call2，
皆有HTTP終局但missing usage object；不得算零。相同prompt的不同付費請求保留multiplicity，
reasoning不重複加總。成本可恢復不代表中斷solver已完成，62/10/0與STOP均不變。
Stock為20 records/2 official-correct/0 wrong，one-shot為20/3/0，feedback為22/2/0且2個
parse failure；沒有provider failure或unexplained new official-wrong。完整task comparison
沒有相對Stock與matched one-shot的重複跨題族Feedback gain；四個未完整task group保持
unresolved，不當成失敗。

因此本輪決定是**stop、不可自動擴大**：這是all-slot-accounted但evidence-incomplete的
limited result，不是complete negative science result，也不是positive gain。任何後續
completion/pivot都要新的prospective admission與fresh host qualification。完整核帳與出版
結果在 `cpachecker-experiments/reports/issue269-fixed-panel-20260921/successor-final-audit.md`；
plan/runtime/source commit hashes維持原凍結值。

## #269 後續：context 落地不等於 reference adequate（2026-09-21）

重查固定 packet：12 個 task groups 中 8 個完整（2 controls 加 6 development failures），
4 個不完整為 t03/t06/t09/t12；不能沿用交接中過期的「3 組」。`reconcile_cost.py`
和 `cost-reconciliation.json` 保留全packet的165 observed加2 unknown usage對帳。

兩次 sorting Feedback 都在 refinement 2 取得 N32/i，接受並注入 native
`c:a[i-1] <= a[i]`，帶有 CE history 與前輪 outcome，最後仍 timeout。因此舊
FIRST_SPURIOUS 沒看到 N32 的診斷不能單獨解釋新政策。t05 array_init_pair_symmetr2
的首輪陣列式因 selected occurrence 缺少 state 被 native SSA/pointer-target guard
拒絕，兩次 Feedback 第 2 輪都已注入 N59 `c:c[i] > 0`，仍 timeout；不能當模型
沒有提出陣列關係，也不能為此刪掉 state guard。

t05 的 source-level reference 是初始化 prefix 的 `-100000 < b[k] < a[k] < 100000`，
接差值 prefix `c[k]=a[k]-b[k]`／`1<=c[k]<=199998`，再保留全陣列正值到 assertion
loop。固定 N=100000 的數學歸納與 120 個有限範例分開記錄；尚未建立可用的 production oracle
map，不宣稱 solver 證明或 backend ceiling。下一步只核對既有 initial-predicate 路徑能否
忠實表示／放置此 reference，不能自動追加 draws 或泛用 translator。
原生 PredicateMapParser 已接受 local SMT assertion map 並交給 fmgr.parse／makePredicate，
不走 production prompt／native C 限制；但 parse failure 及無效 node 可只留下 warning，
正常 exit 不足以驗收匯入。

後續有限 importer 資格已實測：相同固定 runtime/JDK/MathSAT5 5.6.15，t05 真實 CFA
heads 29/53/59；scalar control 各 3 bindings 並 SAT。量化 prefix map 在 makePredicate
需要的 uninstantiate 階段拋出 `Symbols can't start with the "'" character`。独立
stage check 確認 raw SMT parse 成功，但 JavaSMT Mathsat5 visitor 把 bound k 呈現為
free name `'k`，CPAchecker 的 visitFreeVariable 重建名字時遭拒；QuantifiedFormulaManager
亦回報 `Solver does not support quantification`。這是可解析文字和可用 consumer
representation 的區別，不能靠刪 apostrophe／跳過 uninstantiate 假裝修好。

兩個 bounded standalone JVM、僅 1 次 scalar SAT query、0 CEGAR／模型請求；原始
map/stack/hash 皆保留。此量化 reference route STOP，不跑長 reference/control pair，
也不自動換 solver／建 quantifier framework。這不證明所有有限 predicate basis 皆不可行，
不構成模型能力或整體 backend ceiling 結論。相同目錄的 `reference-import/RESULT.md`
和 `check_results.py` 是決定性證據；前面的 production oracle unknown 已縮小到此具體限制。
證據及可重跑 checker：`cpachecker-experiments/reports/issue269-feedback-diagnosis-20260921/`。

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

# 歷史 hard218 的兩個成功案例研究（2026-09-08）

#208 的完整凍結配對結果為 Stock 16 correct、Augmented 18 correct，官方 wrong 都是
同一組 11 題；兩個新增正確題為 `c/systemc/token_ring.06.cil-2.yml` 與
`c/eca-rers2012/Problem14_label45.yml`，Stock 均 timeout。這是單輪觀察，還不是
可重現的 LLM 效果，也不能歸因於 context 或某次 prompt 改動。

使用者明確指定下一主線：兩個成功案例要專門深挖「為何可解、做對什麼、何種程式特徵」，
並平行研究失敗案例。#224 承接這兩個目前 cohort 的案例，連到 #105/#139 的方法；
#225 做分層失敗 census 與相近案例比較；#223 保留 stream/parser telemetry 修正。
這些不是歷史 HH2012/nested9 研究的替代，也不把先前的因果結論自動套到新題。

目前觀察：token_ring 單次 response 有 10 個 candidate items，展開為 14 個
validated/injected head bindings；Problem14 有 12 個 items/bindings。
前者包含 token/local 關係與排程狀態，後者包含數值切點 -43、11、80 與離散狀態。
這些是待驗證機制的線索；接受、注入、數量和 solve 不能直接標成 usefulness。
先完成 source/prompt/response/trajectory 對照，再用原始動態 first_spurious 路徑，
比較 full-response 與 suppressed-injection，先群組移除再逐項移除。後續 trajectory
自然改變屬於實驗結果，不能要求它永遠等於原始 empty trace。

# 歷史工作佇列（2026-09-21優先序已被上文取代）

- #170（擴建計畫已取消，實作/證據保留）：當時的新主線是把 proof failure 編譯成 abstraction vocabulary 的 CFA-native precision
  compiler。MVP 由 exact `ARGPath`、native `AssumeEdge` formula、`EdgeDefUseData` 與 exact
  loop-head placement 產生 `(antecedent_formula, consequent_head, preserved_variables)`，只走
  `PRECISION_ONLY`。完整研究階梯含 symbolic transport、join-aware placement、semantic
  preservation、recurrence compilation、proof-directed optimization、learned residual passes
  與 multi-backend lowering；詳見 `precision-compilation-issue170.md`。
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

單題機制claim仍需其對應的consumer/counterfactual證據；主線pilot先固定比較規則，
不以所有題先達consumer-positive作前置。凍結gold-free prompt、scoring與stop rule；
generation、validation、trajectory、verdict、cost 分開記錄。完整生成 response 必須原樣 replay，
不能只挑成功 atom。正式 model call 必須重用 Java `PredicateProposalClient` transport；參見
`vguide-experiment-transport.md`。研究規格以 GitHub Issues/Wiki 為準，artifact 在
`<remote-home>/cpachecker-experiments/`。
