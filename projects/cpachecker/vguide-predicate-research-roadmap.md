---
title: CPAchecker VGuide predicate 研究主線
project: cpachecker
scope: project
tags: [vguide, predicates, cegar, nested-loops, research]
status: active
created: 2026-08-26
updated: 2026-09-27
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

主協調證據在 `cpachecker-experiments/reports/three-route-fullset-20260926/`，各路線為 `issue281-fullset-20260926/`、`issue282-fullset-20260926/`、`issue287-fullset-20260926/`；#280 與 Wiki Three-Route-Fullset 維護目前驗收。三條路線各自已設獨立 goal；前輪結果是起點，不是 goal 完成證據。9/27 使用者要求繼續並降低 subagent 成本，後續依任務使用 Luna／Sol／Terra、只在必要困難推理使用 Astra；不恢復大量高成本 owner 或重做已通過審查。平台 usageLimited／blocked 狀態不能用 update_goal 改回 active，也不能為此假標完成。

9/27 reducer v1 的完整官方 unreach-call 28 題原始配對為Stock 2 solved／26 UNKNOWN、路線11 solved／17 UNKNOWN，觀測10new／1lost。但後續整服務核帳發現Stock有19筆超600 CPU預算的UNKNOWN，不能把這些當合格等預算baseline；原始數字與成本保留，嚴格配置效果須補測。25題在已審signed fragment中使用候選，3題保留Stock；另22個其他property仍在全50 inventory。v1沒有LLM。10個觀測new的Stock原因為5 CPU耗盡、2 MathSAT interpolation error、3 cgroupOOM；lost是rangesum05，抽象反例接不回原C而UNKNOWN。見`three-route-fullset-20260926/issue287-v1-family-independent-audit.json`及`issue287-fullset-20260926/WHOLE-SERVICE-COST-AUDIT.json`。

### 9/27 完整性與歸因更正

**舊排序 TRUE 不能外推成完整初始化語義的原 C 證明。** Frama-C31 一般 `-wp-rte` 不會自動包含全部局部變數初始化檢查；須明確設定 `-rte-initialized` 並核對所有實際函式。原兩份成功完整LLM plan逐byte不改，補此設定後bubble為101/105 UNKNOWN、selection為168/178 UNKNOWN，未證4／10個義務全屬陣列讀取初始化。它們不是C反例，也不表示原invariants已被否證；但原6題F1的2TRUE/4UNKNOWN和hard218單例增量只保留為legacy configuration結果，完整原C proof gain為待定。新16题consumer與兩safe修補必須保留這些義務。#282 comment5852367214、#280 comment5852383144及Wiki Three-Route-Fullset已公開更正；下方舊80/80、84/84、127/127與99/99歷史證據亦不能無條件外推完整初始化語義。

- **模型收益需相同策略的無候選對照。** reducer v4同時改人工gate precision、CPU分配、UNKNOWN後exact fallback；25題15TRUE/5FALSE/5UNKNOWN、完整family28為21solved，對Stock的19個觀測new含不合格baseline，且不能全歸因bounds，更不是LLM增益。五rangesum各用不到舊38CPU，主要差別是舊UNKNOWN未進exact分支。v5/v6對v4只衡量替代人工，已另完成同策略去除初始precision的full25 F0，見下方配對確認。hard218的v6是5solved／2未解／211未套用；「solved」不能直接改寫為對Stock「new」。
- **研究檔案adapter不等於production輸出接通。** #287原始模型SMT predmap確經既有initialPredicates消費，但function-scoped qualified symbols、虛擬PREDMAP_FILE carrier不是native-C expression或真CFA head。不要以重新標籤冒充原resolver整合；優先直接讓模型輸出已支援native C與真head，先核encoding/placement/實際precision，再看解題，無證據不造SMT→C transcoder。
- **完整模型收據才能採納成本內證明。** #282舊採納器對缺receipt或model timeout仍可能保留TRUE；外部overlay核完整contiguous draw/request/response/plan ledger、一對一hash、正常退出、usage自洽與總額。2正例＋7負控制獨立複驗PASS；缺或矛盾usage保留unknown，不能當零或已知subtotal算全成本。此修補只關乎完整性，不會補齊上述初始化語義缺口。見`three-route-fullset-20260926/issue282-f1-boundary-followup.json`。
- **byte相同C仍可能有不同嚴格request key。** #281重新匯出使structured trace中的source.file路徑改變，原cache無法命中；v2失敗保留。明示path-only等價localhost fixture的v3保留原回答byte，actual Java request逐byte核對，完整13題5TRUE/8UNKNOWN，5次draw有45/45precision members；其中3題轉換直解、2題用LLM回答，double第3份未用。這是capability/developmentpolicy，每題whole-service600CPU/900wall內，但原6次生成12226tokens在外且generationCPU未知，不冒充含生成正式portfolio。見`issue281-native-family-v3-independent-audit.json`。

9/27 full25 LLM與F0比較及全部gain確認已收齊：F0為7TRUE/5FALSE/13UNKNOWN；v5原答12TRUE/5FALSE/8UNKNOWN；v6修補15TRUE/5FALSE/5UNKNOWN，其中19題原樣重用v5、6題另跑。相對F0，v5新增5、v6新增8、均無退步；sum10/40修補自主，sum60用過人工概念提示。完整8個NEW/LOST聯集在Mazu另做24 logical／19 actual同機配對（5份v6與v5逐byte相同明示重用），F0八題均UNKNOWN，v5五TRUE三UNKNOWN，v6八TRUE；1181凍結依賴及每筆原始服務成本核對通過、實跑合計6697.795 CPU秒。全8效果重現，但不是fresh LLM regeneration；離線生成成本另列，不宣稱端到端加速。hard218的LLM增量僅原答2／修補3，不能外推family的5／8。見`issue287-f0-complete-independent-audit.json`、`issue287-llm-complete-independent-audit.json`、`issue287-paired19-complete-independent-audit.json`。

### 整體成本、benchmark語義與production接合

- **worker CPU不是整服務CPU。** `RUSAGE_CHILDREN`可漏啟動器、本身和未reap的timeout子孫；`prlimit=600`亦不保證整服務不超600。#281/#287以原始systemd/cgroup總額重算，舊超額UNKNOWN保留且不作等預算baseline；新批次沿用既有599.5 CPU／899.5 wall整cgroup watchdog，最終仍嚴格核600／900、global dependency drift與完整終局。缺CPU不能寫零或把known subtotal寫成total。清理只作用已驗身分的自有cgroup，須涵蓋另開process group的子孫。#282 Docker外層`time`也曾漏掉timeout Alt-Ergo；歷史成本只在有證據時給完整保守上界，未能界定者補測，不能事後捏造精確值。見`issue281-whole-guard-review.json`、`issue282-positive-cpu-upper-bound-independent-audit.json`。
- **SV-COMP配置成功語義要隨benchmark invocation給定。** 原`ifcomp`的N=2反例靠三次malloc皆NULL形成alias；官方假定malloc／alloca成功，generic PredicateCPA預設卻容許NULL。既有`cpa.predicate.memoryAllocationsAlwaysSucceed=true`即可對齊，官方svcomp配置已有；不需改library全域預設或benchmark。單選項控制使錯FALSE消失成MathSAT插值UNKNOWN，沒有新增證明。完整226題逐source／config／parser轉換路徑核對：38題可能受影響，188題無allocator且不經隱式產生分支；只重用來源與整體預算均合格的49筆，其餘633筆另凍結，完整682分母保留。見`issue281-ifcomp-stock-triage.json`、`issue281-allocation633-freeze-independent-audit.json`。
- **native與SMT的合法head集合不同，prompt不能混用。** PR293（merge e8ffb7aace30）保留SMT trace heads，另列native-eligible heads，並提供actual Specification與machineModel；最初把全部head換成native集合會退化SMT，已以控制修正。64項focused tests通過，未改舊實驗runtime。Live avg first trace已有N15可接受ret–i bounds；與offline成功請求的abstraction states／block formulas／var contract／local precision逐值相同。N42拒收與模型未提出ret–i是兩個問題，無證據重寫resolver；固定陣列32-bit C sum也不能直接當64-bit累加式。此PR本身尚未證明benchmark增益，FIRST_SPURIOUS只呼叫一次的schedule亦未因此改變。見`issue287-native-context-mechanism.json`及PR293 publication receipt。
- **BMC 開啟 VGuide 旗標不代表會呼叫模型。** 原 `bmc-linear` 的固定 bound BMC 不走 Predicate CEGAR spurious callback。**INTEGER native consumer 已有實際資格，但仍須量測收益。** 原 fixed-bound BMC 不會觸發 Predicate CEGAR 的 native LLM hook；現成 `predicateAnalysis-linear.properties` 可接同一 consumer。max40 的 F0／FIRST1 完整原題都 UNKNOWN，FIRST1 的 10 條 C 候選形成 12 個 PRECISION_ONLY head bindings，全部注入、下一輪 precision 讀回並出現在後續抽象；一次 live call 7356 tokens。這不是已證不變量或解題增益。59 筆 refinement 只有一次呼叫，後58筆因原 FIRST1 round 上限略過；後續政策研究需區分內容不足、後續回饋與接點失敗。當前 checked BMC 與此 CEGAR 都是 `ignoreExtractExtend=false`，不能誤用已退休的 true diagnostic 當現行配置差異。 見 `issue287-integer-native-r4a-independent-audit.json`、`issue287-integer-native-failure-analysis.json`。

- **完整輔助 WP 集合擴充為23題，全部五個正例已原答確認。** 原20題外，新加入的完整三題比較只有 heapsort 轉 TRUE，得到 F0 23 UNKNOWN、E3 5 TRUE／18 UNKNOWN；現行 consumer 可接的九題 sorting 全部已測。heapsort 是純量索引安全、沒有陣列排序資料，不能說成 permutation 證明；其完整義務170/170，含47 init。其餘四個正例及完整初始化模型限制沿既有記錄。所選206份模型收據完整927317 tokens，包含失敗抽樣；整個研究431次中另有三份 usage unknown，不能混成無缺成本。九個 safe UNKNOWN 都涉及 VLA，pr3另有普通未初始化 MINVAL。此路線仍非 native CPAchecker 效果；共同 hard218只含前三個正例。 原四題 init／find／bubble／selection 的完整義務為60/60、93/93、151/151、214/214；原方案確認4TRUE、0新呼叫，追加CPU上界12.759137秒。新增 heap 原方案確認170/170，0新呼叫、CPU上界4.895274秒。不能把確認成本當生成成本或端到端加速。見 `issue282-full23-ledger-independent-audit.json`、`issue282-heapsort-confirmation-complete-independent-audit.json`。

- **scalar nondet修復不能直接推到VLA。** Frama31的`__fc_vla_alloc`由parser產生宣告，沒有可inline的body，WP31也未實作allocates義務。任意initialized pointer不能補freshness、size、validity、nonalias及lifetime；wrapper或Typed cast開關不會證成這些前提。現成Eva有VLA記憶體模型，但尚未證明能向WP交付完整premise；Frama33的相關return/init規則與31相同，不能靠「升級應該就好」換frozen runtime。見`issue282-vla-minimal-path.json`與`issue282-upstream-source-check/REPORT.md`。

- **合法的nullable declaration不能直接取type。** parser允許未解析CIdExpression沒有declaration；ArrayAccess visitor訪問function name後直接getType，12官方案例因此NPE，實際id是__builtin_bswap16/32/64。PR291（merge c6f6a5c15b）在全CFA、任何simplification之前保守拒轉並保留同一原CFA；不是跳過未知id後繼續轉換。summary edges、calls/arguments/returns/initializers及package callers已查，generated ids有concrete declarations；12前後identity控制＋safe/unsafe原題控制全部通過。PR289同樣保守處理nested array base（merge abe33c0664）。兩修補不宣稱新增語義支援或benchmark gain；原frozen research runtime不換版。見`issue281-fullset-20260926/unresolved-id-fix-20260927/CALLER-PROOF.md`及兩PR publicationreceipts。PR288全域不變bound＋strict-endpoint安全修補也已合併125a5855af。

- **正確的 class archive 不保證研究封裝可執行。** 自訂 Ant wrapper 若直接 import librarybuild-jar 而漏掉 canonical `jar.excludes=""`，模板的 `**/*Test.class` 會刪除名字以 Test 結尾的 production nested classes。此案遺漏七個 production classes，包含 `AutomatonBoolExpr$BoolBinaryTest`；自檢沿同一錯誤 filter 而錯誤自洽通過。保留失敗包與全部成本；最小修正是沿官方空排除設定重包原已測 archive，核對全部 5549 個 class bytes，再真正執行 safe／unsafe、overflow／INTEGER 與 native callback smoke。不能把正確重包宣稱為乾淨 source rebuild，也不必修改本來正確的 production build。
- **direct SPC 的 property unknown 不代表規格不存在。** `Specification.getProperties()` 只包含從 `.prp` 解析的 Property；直接 `.spc` 仍載入實際 automata，CPAchecker 也實際消費它們，但以 getProperties 建 prompt 的既有橋接會顯示 unknown。不能從檔名或 benchmark expected label 猜造 LTL，也不能據此宣稱這造成模型失敗。可另列實際 specification identity，語意與 parser metadata 應分清。
- **零長度 VLA 顯示原 property 與 WP 前提的差別。** 原 insertion_sort-2-2 的 SIZE=0 可到 VLA 宣告，Frama31 前端產生的 alloca_bounds 要求正長度，與 C99 6.7.5.2p5 一致。當前 WP 協定的 defined-execution 前提強於單一 unreach-call；不可刪 guard、補 SIZE>0 或假稱官方規則已豁免。配置初始化、存取與 frame 仍是獨立義務；timeout 的部分 VC 日誌不是完整 remaining-goal census，也不是原題不可證的證據。
- **固定回答 replay 要核對全局有序序列。** 現成 cache ordinal 是每個 request hash 自增，不是全局次序；A,B 與 B,A 都能各命中 ordinal1。因此比較 phase／refinement／call kind／hash／ordinal／response 的有序序列及未消耗、額外呼叫，不能只數 cache hits。replay 無新 HTTP terminal evidence，usage 是歷史生成成本。輸出目錄不直接進 request body，但 source path、CFA／SSA、MathSAT `.def_N` 與 native/history context 會進入；沒有跨跑完全相同的保證。miss／分析分歧應保留為確認未完成，不重新綁 hash、偷抽新答案或算成原策略 LOST。
- **Newton fallback=false 仍可回插值。** 現成 Newton BLOCK 可保留 pointer-aware heap 及共同 native hook；UCB 在同配置下因 WP converter 為 null 而不適用，不能關 pointer aliasing 改原語義。Newton 的 repeated-counterexample 分支無條件回 interpolation，實際 array_tiling_tcpy 已觀察此 fallback 後的 ie-local error；entry counter 亦不能證明完成一次 refinement。`solverQeTactic=NONE` 時仍可能出現 constructor 的一般 QE 警告，應核 actual UsedConfiguration 與原碼分支，不按警告文字判選項未生效。完整五題新對照共十列皆 UNKNOWN：四題 Newton 在通過初始插值障礙後耗盡整題預算，tcpy 則回插值失敗。整服務合計2426.368 CPU秒、零模型呼叫。INFO「refinement #」位於 strategy.performRefinement 前，最後一筆不等於最後一輪完成；後續下一輪才證明前一輪返回。這些都是作用邊界，不是全family新增能力結論。


- **新版 native 線上全25題已出現增益，確認仍在進行。** 同runtime完整75列：F0 7TRUE/5FALSE/13UNKNOWN；FIRST1 12TRUE/5FALSE/8UNKNOWN，新增5/退步0；EVERY1_HISTORY 15TRUE/4FALSE/6UNKNOWN，新增8/退步1。首次組新增avg10/20/60與sum10/20，每輪組再加avg40與sum40/60，但失去rangesum20。兩策略在共同hard218的增量都是avg10、sum10、sum20三題，不能把family的5/8全部灌入218。75整服務合計17580.867 CPU秒、1289依賴與來源完整核對；不是PR293單獨效果或後續history必要性證明。FIRST1 20次完成呼叫117027tokens；EVERY132次HTTP開始、131完成，已知1073110tokens，另一次用量未知，總額必為unknown。全NEW/LOST確認未完成；缺第17份回答的rangesum20不能造replay，另做fresh同策略復驗亦須明示生成變異。見 `native-latest25/ANALYSIS-r5.json`、`issue287-r5-root-full75-cost-check.json`、`issue287-r5-complete-independent-audit.json`。
- **CPU cap不限制模型等待，前段可擠掉整個fallback。** rangesum20在EVERY策略用895.331wall、33.950CPU後成UNKNOWN，F0/FIRST1原為FALSE；16完整呼叫140781tokens，最後第17次只有HTTPstart。前段58CPU限額沒有對應phase wall，實際可使用整題895wall，未到後段原題驗證。現成`limits.time.wall`只請求shutdown，不等於外層硬timeout；應先沿現有父wait/killpg研究保留後段wall，不靠任意小callcap。修正尚未實測，原loss保留。
- **殺掉分析子程序不保證服務已在期限內結束。** max40 INTEGER EVERY history診斷有26個HTTPstart、25完整回應275183已知tokens，最後一個回應/usage未知；結果whole273.065CPU但903.162wall，最終guard正確列budget-invalid，不能作合規UNKNOWN對照或新增收益。worker在checkpoint之後還會核1129個依賴及收尾，而watcher只殺children；整服務成本仍須含其自身。原報告與calls保留、不改900上限；後續必先修正/驗證生命週期界限，不能拿worker896.884wall假稱whole合規。
- **native「pure C」並不接受所有純布林寫法。** e8ff的ProposalPromptBuilder明列禁止`&&`、`||`、`?:`，EclipseCParser.parsePureExpression亦拒收；不能只看NativeCExpressionEncoder就推論任意布林式可用。固定40個元素在數學上可有限展開prefix relation，但既有比較式0/1加總僅來源上可表達，尚未通過實際consumer或證明測試，跨call資訊保留也未決。完整複合式若接受會成為一個precision predicate，不會自動原子拆解或假設為真。H的25完整回應354候選最長23字元、僅出現少數固定索引，沒有完整有限prefix關係；不能因此直接否定LLM或宣稱必須重寫量詞引擎。見 `issue287-finite-prefix-native-feasibility.json`。

### 已驗證的 consumer 與算術限制

- **初始化 relation 已有、consumer 仍可能缺回傳效果。** WP31 的 scalar nondet call 經 generic assigns／value equality，未建立需要的 initialized destination；既有開關、scalar contract 與一次 Eva 預算延伸均未補齊。#282 最小可用接法是獨立 trusted SV nondet model TU＋既有 inline：fresh local 透過只寫該 scalar 的無數值限制 primitive 初始化，再 return；原 C tokens 不變。int／uint bounds+init、兩次呼叫不能假定相等、普通未初始化仍拒收等五控制通過。在同 TU 下，bubble 舊 plan 117/121 UNKNOWN，首份原始 LLM repair plan 131/131 TRUE、含28init義務；source/rawresponse/plan、完整wrapper/caller/init census與實際採納guard獨立重算PASS。這是明示模型前提下的seeded capability，不是純註解、不恢復舊2TRUE的完整集增益，也不宣稱新的含生成600/900成績。見`three-route-fullset-20260926/issue282-nondet-model-review.json`。
- **native原題成功仍須區分offline回答和live prompt。** #287 avg60/sum60的新source+真head metadata回答24條全被既有native-C encoder接受／注入，資格檢查24/24exact precision members。原回答重播進checked overflow→INTEGER原reachability均TRUE，whole service29.106/24.426CPU；原2calls13973tokens在預算外。沒有必要SMT→C transcoder，但不能叫production線上全流程完成。後續首個Java live call確為live_recorded，12條中4條head_not_on_trace拒收、8條接受，內容缺研究prompt得到的ret-progress；source/head/CE/prompt差異須實際對照，不能歸因native語法或拿離線答案冒充live輸出。原production FIRST_SPURIOUS完整25後續已終局，7TRUE/5FALSE/13UNKNOWN，對相同政策F0新增0/退步0；全25整服務8442.521CPU，20次livecall/128451tokens，20prompt/round/cache逐byte与usage對齊、無未完成prepared事件。13未解分成8gate未證、5gate已過但INTEGER/exact未證；更多gate回饋不會直接作用後5題。跨host且續跑有外部競爭，不作速度比較；provider CPU未知。最新runtime的每輪回饋策略須另外固定並完整比較，不以原策略負結果否定全部LLM路線。見`issue287-native-targets-independent-audit.json`及`issue287-native-live25-complete-independent-audit.json`。
- **cast UF 不足以保留 unsigned 語義。** `ignoreExtractExtend=false` 的 INTEGER 路徑，在同寬 signed/unsigned 比較與 unsigned wrap 兩個原 C 反例給錯誤 TRUE；精確 BV 為 FALSE。既有 `cpa.predicate.encodeOverflowsWithUFs=true` 使這兩個控制恢復 FALSE，應先測現成功能，不必重寫 resolver。但這只是兩個控制，不能推論通用 soundness；no-overflow property 也不能代替明確 cast、unsigned 或 pointer 操作的接合審查。原 signed fragment 的排除條件仍有效。root 已逐份核對六個 raw logs／source hashes，見 `three-route-fullset-20260926/issue287-signedness-independent-audit.json`。
- **排序 importer 與模型失敗要分開。** Clang AST 同時列出 implicit builtin 與原 `abort` 宣告，機械產生兩份 ACSL contract 會先導致 parser error；排除 `isImplicit` function declarations 後，同一批原模型回答才進入真正 VC 檢查。安全 selection 的模型原先在 swap 後仍引用 `a[s]`，一次原始失敗 VC feedback 後自行改用當下 `a[i]`；完整原 C、未人工修改的22項模型候選經既有 JSON schema／註解 consumer 得到127/127。root 重建輸出逐 byte 相同，核對44個 invariant initiation/preservation 和原 assertion caller obligations。這是已曝光研究題的 auxiliary backend 證據，尚非 CPA 或完整集合效益；見 `three-route-fullset-20260926/issue282-selection-independent-audit.json`。
- **整體成本不能只看最後 checker。** gate、INTEGER 和必要 exact-BV validation 均計入同一600 CPU秒；各 phase 的 `prlimit` 是單 process 限制，還須核對 `RUSAGE_CHILDREN` 累計與正式 cgroup 總 CPU，再採納 verdict。證明義務全過和 consumer 實際使用只證明該例可用，新增解題仍需配對 baseline 與完整分母。
- **native 轉換成功與關係足夠是不同問題。** #281 double/triple 各三份原始 live LLM 回答，53 個 predicate/head bindings 全部通過 native C consumer、實際注入且 exact uninstantiated precision membership 全中；六次離線 replay 為3 TRUE／3次600CPU UNKNOWN。raw回答、來源、validated/injected multiset與terminal logs已獨立核對；pipeline重排使舊order-sensitive介面旗標false，不等於丟候選。失敗三份不能再歸因resolver拒收，仍需關係充分性／refinement診斷。這是兩題機制證據，不是每題一次600CPU的best-of3或完整集收益；見 `three-route-fullset-20260926/issue281-native-six-independent-audit.json`。
- **印出 verdict 不等於子程序完整成功。** reducer研究wrapper曾只看log，可能把印TRUE後hang到inner wall cap的子程序採納，且外層wrapperexit0掩蓋逾時。新正式兩臂啟動前以`exit==0 && !wall_timeout`約束effective verdict，raw verdict仍保存；gate與INTEGER／exact fallback都消費effective verdict。12個真child的正常／nonzero／timeout控制及獨立複驗全過，原frozen版本保留。見 `three-route-fullset-20260926/issue287-v4v5-r1-local-review.json`。這是結果完整性修補，不能冒充signed fragment健全性的通用證明。
- **整個 cgroup OOM 會連 wrapper 一起殺掉。** 因此缺少terminal JSON不必然是infrastructure failure；有systemd明確`Finished with result: oom-kill`且launch/source證據吻合時，應記資源UNKNOWN，從服務紀錄取得可觀測CPU/wall，unknown exit保持null，不虛構-9或0。沒有明確終局證據仍是infra；原missing-terminal紀錄保留，用獨立reconciliation接合，不覆寫原始失敗。修正collector的fault controls與獨立審查PASS，且新Stock sum20實際OOM成功分類，見 `three-route-fullset-20260926/issue281-resource-runner-independent-audit.json` 與reducer完整family audit。
- **候選都進入precision，仍可能少一條關鍵bound。** 原triple第一份LLM回答已全部native消費但600CPU UNKNOWN；保持其餘7條不變，只追加人工診斷`c:__array_index_a <= 300000`後TRUE約2.5CPU、8/8 exactmembers。這支持該配置的關係充分性解釋，不是通用數學必要性、resolver修復或LLM自行發現。見 `three-route-fullset-20260926/issue281-single-bound-independent-audit.json`。

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

### 9/27 接續的已驗證邊界

- native75 的新增／退步集合已完成22次嚴格固定回應重播（48份回應、零新HTTP）；原觀測皆重現。原rangesum20最後一次模型請求未完成，另以全新draw得到UNKNOWN，仍只有overflow gate執行，895.707 wall、50.668 whole CPU。兩次全新生成都出現前段API等待耗尽wall、下游沒開始，支持研究最小wall reserve；不能把新draw冒充原完整重播。
- Newton BLOCK＋native repeated LLM 的完整5題配對共10次皆UNKNOWN，4824.549 whole CPU，37次完成模型呼叫、280152 tokens。各模型臂候選確實注入並出現在下一輪precision，04仍終止於ie-local interpolation failure，其餘4題耗盡CPU。這5題原陣列抽象census全UNCHANGED、沒有轉換array/loop。候選送達precision不等於所需跨loop前綴關係已在抽象狀態成立；不能以更多候選或readback當作突破。
- ArrayAbstractionAlgorithm delegate 的外層等待迴圈必須尊重既有analysis.stopAfterError；多次delegate結果須合取AlgorithmStatus。原控制忽略stopAfterError且覆蓋先前status，PR295以3個回歸測試修正，舊實作mutation均失敗；合併47cc91c。imprecise CFA重新驗證原C時的status替換是另一個語義，不可一併改成合取。075同編譯器配對實測已完整收帳：只差wrapper class的舊臂UNKNOWN599.577CPU、新臂正確FALSE66.115CPU，whole-service預算、來源與final closure皆合格；這恢復反例，不是LLM增益或整體速度結論。原075 LOST紀錄保留；獨立重複再次得到正確FALSE，62.991 CPU／40.211 wall，完整收帳與實際釋放通過。
- 建立同編譯器控制時，來源相同不保證舊jar逐byte相同；本次重新編譯的兩臂只有外層wrapper class不同，與舊凍結baseline分開保存。runtime透過config/lib symlink執行時，closure須涵蓋實際alias檔案路徑，僅hash原resolved paths不足以攔截alias改指；使用既有全域hash驗證即可，不必每row重複。
- wall-reserve follow-up 的全25題新鮮生成得到18題正確，歷史EVERY為19題；相對F0新解6／退步0，相對EVERY救回rangesum20但失去sum20、sum40，不能宣稱整體改善。兩個lost gate均先耗盡原有58 CPU（約352／362 wall），沒有觸及新增597 wall截點；其後exact fallback耗盡whole CPU。129 HTTP starts中108成功、17個503、4個未終局，已驗證890973 tokens，總量未知。新draw的服務失敗與生成內容仍混在一起，不能把這兩個loss歸因於wall reserve，也不能把503或未終局呼叫的未知費用記為零。9題NEW／LOST聯集需完整確認；後續draw不能刪掉這次觀測。
- 固定回應確認的順序必須來自原始`llm_rounds.jsonl`，不能攤平依request hash排序的`model_ledger.calls`來重建時間序列。本次5題完整draw皆會因這種排序被錯判序列不同；改讀原始順序後，23份完整回應均可嚴格重播，5題TRUE全重現、零新HTTP。成功HTTP以外的503／中斷若沒有完整response cache，就只能另做獨立draw，不能宣稱完整重播。
- 完整集合收尾可對已固定雜湊並完成triage的單一既知native crash保留INVALID，排除涉及該無效結果的比較，同时保留同題其他有效pair。不能把它轉成普通UNKNOWN，也不能讓它永久阻擋其他已完成題目的NEW／LOST確認；完整分母、有效覆蓋與未完成項須分開。052 Stock特例的其他兩臂或任何新題若出現未診斷INVALID／WRONG，仍阻止確認清單凍結。
