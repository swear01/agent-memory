---
title: 原生 C 謂詞接入的純度、scope 與 trace context 邊界
scope: projects/cpachecker
status: verified
updated: 2026-09-23
---

CPAchecker 原生 CParser／PathFormulaManager 可以重用，但一般 statement fragment helper 不等於純 expression parser：x++ 可被 ASTConverter 降成暫存 expression，副作用留在 Sideassignments。外部候選須先檢查 CDT 原始 AST，再確認沒有 pre/post/conditional side assignments；empty include provider 加上 preprocessing 拒絕可防止 fragment 讀取 include 檔案。既有 witness helper 不應為此改變語意。

AstCfaRelation 的舊 active-declaration snapshot 不包含 nearest-shadowing 順序，同名 outer/inner locals 可能並存。PR272（merge c8b7975552468c556ef264ff081f3c8631cf2bb7）改由 C frontend 保存每個 node 的 persistent lexical name→declaration snapshot，經 ParseResult／AstCfaRelation 傳到 CProgramScope.withVariableBindings；此 lookup 沒有 permissive fallback。原始 source name 優先，只有仍可見的 generated alias 才補入；已離開作用域、foreign 或缺少 metadata 仍拒收。native C encoder 只在 C CFA 初始化，其他 frontend 保留既有 SMT 路徑。

使用與 block formula 相同的最後 aligned trace occurrence 的完整 PathFormula（SSA 與 pointer targets）。最後一次 context 缺失時不可退回較早 SSA。若編碼引入新的 SSA／pointer-target state，先報 unsupported context。顯式 c: 標記失敗不得改用其他 grammar 猜測。預期 parse／transfer／argument 錯誤可拒絕；意外程式缺陷不可被 broad RuntimeException catch 偽裝成候選品質問題。

PR268 已合併；初始 84 tests，加上最終受影響 31 tests，共 87 個不同 focused tests 通過。真實 CFA 編碼、型別／signedness、scope、同一 context 與 predicate/local precision 邊界有回歸檢查。一次 exposed-source consumer probe 顯示原 mixed-SMT 字串被拒、人工 c: 關係可插入 precision；其構造 prefix 未驗證可行性，不是模型生成、adequate oracle 或 solve-gain 證據。native-specific rejection 尚不在既有付費 repair allowlist；改變生成語法和 repair-call policy 必須分開歸因。

實際 FIRST_SPURIOUS 消費檢查（2026-09-21，既有 exposed sorting case）已超過上述構造 prefix：相同新 runtime／首次 preselection context 下，保留原9個有效 candidate objects 有10個 head bindings；人工加入原生相鄰關係後有11個，皆為 PRECISION_ONLY。新增關係在實際 N45／SSA／heap context 編碼，並在11次有界診斷中到達 definitionPush、AllSAT 和正常返回；10次有 callback。兩邊仍為600 CPU秒 timeout、各93 refinements，沒有新解題或模型生成證據。診斷預算用盡後的缺少事件不能當作沒有使用。

原生 C 編碼成功不補足數學證明義務：單點 a[x] <= a[x+1] 不等於有界量化的全域排序／跨迴圈關係，head／exit 的 C 存取還需範圍條件。另一方面，PRECISION_ONLY 是抽象分割特徵而非程式假設，不能只因候選非 invariant 就宣稱 verifier 不 sound。換 prompt 後請求 hash 會變；人工搬移歷史 response 必須明說是 intervention，先捕獲新的 production request，再驗證 context／cache 身分，不能冒充新的模型回答或原請求的 cache hit。研究紀錄：reports/issue259-native-refinement-20260921/，父 issue259／103 保持開啟。

另一個已知 mechanism case 的新生成結果不能與上述 sorting 負例混為一談：#151 在同一 8c3c822 runtime 的 cggmp2005_variant 上，先捕獲 actual FIRST_SPURIOUS prompt，確認 contract 有 main 的 lo／mid／hi，再以 Muse 做 control／affine cue 各3次。六份完整新回答皆含 hi = lo + 2*mid 的等價 c: 表達式，原樣經 production validation／PRECISION_ONLY injection 後皆 TRUE（五份1次 refinement，一份2次）；empty response control 在300CPU timeout，既有 affine+bound reference 為 TRUE。這是 native C 路徑可承接模型實際有用輸出的單案例證據，不是新 hard218 gain；兩組3/3平手，不支持 cue 優越性。模型／runtime／prompt 都不同於舊 DeepSeek interrupted cohort，不能歸因為單一 scope 修補或填回舊缺失 draws。外部 cue 生成的 response 對共用 consumer request 的映射須明確保存，不能假稱 treatment prompt 本來就是 runtime 產生。研究紀錄：reports/issue151-fresh-generation-20260921/；單一謂詞族的必要性須另看移除對照，不能從完整回答 TRUE 自動推定。


2026-09-21 的 #271 全量轉換調查在 main `64d5888775a9d99c35c2a26a6dbb4aaccff9da8a` 確認上述保守 shadowing guard 的實際能力缺口：zero_sum4 的 N29 同時保留 outer main::i／inner main::i__1，CFA condition 使用 inner，但 function-wide parser 把 bare i 解成 outer；按 origName 計數又使精確 alias i__1 也被拒。167 份原生 C 回答中共24筆相關拒收（8 bare i、16 alias）。N35 離開 inner 後 outer 可用、inner 正確拒收。最小下一步是 head lexical declaration identity，不能刪 guard、以最大 source line 猜 nearest scope，或只改 prompt 名稱。這是修補前的表示能力缺口；PR272 已修，是否增加解題須另看固定回答對照。

同次調查把所有接受公式逐一 uninstantiate 後，與保留的實際 post-injection local precision 比較：1,411/1,411 exact members；221筆沒有獨立結果的 head/raw pairs 全為 whitespace／canonical formula aliases。這不證明各公式有後續 solver 消費或解題因果。7個原生 C 控制例（promotion、signedness、array point、2D、pointer、remainder、bitwise）與 CFA edge 等價，支持重用已有 encoder。synthetic SSA fixture 的 CType 要依原生 pathformula 慣例移除 scalar qualifiers；const CType 失敗不能誤判為真實模型轉換缺陷。

現成 ArrayAbstraction 不能假定與 VGuide native C plug-compatible：原始 array_init_pair_symmetr2 有3 eligible arrays／0 eligible loops，回 UNCHANGED；TransformableLoop 以空 ValueAnalysisState 找 constant comparison，無法直接承接 i < N。簡單 for／while 控制例各轉1array／1loop、PRECISE，但轉換後全部23／22 nodes 查不到 AstCfaRelation scope；CFA transformer 沿用舊 metadata，source prompt 仍讀原 source。需要另做 eligibility、scope／source context 與 witness 表示資格檢查，不能把改 JSON／開 config 當作量化關係已接通。研究紀錄：reports/conversion-gap-audit-20260921/，issue271；沒有 verifier／provider run。

PR272 的重要 frontend 邊界：compound block、for loop 與 statement expression 離開 scope 時，successor CFANode 可能沿用 block 內已建立的 node。只在 node 建立時存 snapshot 會把 inner binding 留到外層，必須在 leaveBlock(successor) 後刷新該 node 的 lexical map；舊 active-variable 集合不必改動。原始 source spelling 必須先於 generated aliases，避免 x__1 與另一個 source variable 同名時綁錯。最新 head 6ea3106 的 82 個 focused tests、Checkstyle、24/24 原拒收 binding 的 real-CFA/synthetic-SSA 檢查及7個原生 CFA 等價控制通過。Default build 與 pristine base 同樣受既有 IdentityHashMapUsage 警告阻擋；qualification 僅關閉該一 inherited check，保留其餘 Error Prone 與 -Werror。

#271 後續固定回答對照已完成：zero_sum4 的 Stock／原 consumer／修補 consumer 各兩次、每次600CPU上限，六次皆UNKNOWN／timeout，predicate refinements為18／4／2（另列CEGAR counter19／5／3）。四份完整歷史回答在四個augmented runs的request/content hashes皆一致，16個response reads全為真replay、0新增provider calls。每次原版4個shadowing拒收拼法恢復為2個distinct formulas，actual injection34→36；跨四次共140/140公式經原生parse／uninstantiate確認actual precision membership，原版是修補版子集，新增公式都綁inner main::i__1。這修復轉換能力但沒有新增正確解題，不能把refinement較少當速度或解題收益；一題固定回答不支持population結論。reports/issue271-native-shadowing-20260921/保存獨立runtime、protocol、原始logs與read-only checker；#269停止格未改；其後PR274–277已修復#271選定的其餘轉換缺口，見vguide-predicate-validation-contract.md。

#273 的原生熱路徑修補只略過 native-only candidate batches 用不到的 legacy array template extraction。既有 encoder 的 lexical scope、最後 aligned PathFormula、SSA／pointer-target guards 沒有放寬；含 legacy 的 mixed batches 仍抽 template。真實 CFA 的 native-only／mixed 控制分別0／1次 block serialization且接受結果正確，最終110 focused tests通過。167份保留回答的parser結果全相同；沒有新verifier或provider run，不能以減少內部工作推定solve或timing收益。


2026-09-22 #278／PR279 證明陣列 relation 可走現有 scalar witness 抽象，但必須區分表示與生成：原始 array_doub_access_init_const（N=100000）經修補 ArrayAbstraction 自動匯出 C，再 normal reparse 取得真實 scope，人工投影既存 slot06 的 forall-prefix 關係到 JSON+c: 後 TRUE 兩次、各1 refinement；保留相同三個 bounds 但移除 prefix 則60CPU timeout兩次、各86 refinements。14/14公式在actual precision；小型原始 safe／odd-write-1 controls為TRUE／FALSE。這是原始程式自動轉換＋人工reference的單案例機制證據，0新模型呼叫；不是raw prose自動轉換、fresh LLM generation、整體速度或population收益。證據在reports/predicate-representation-goal-20260922/，只讀check.py。

修補關鍵：多subscript初始化loop不能粗略collapse，須保留迭代及每次write，並依arbitrary tracked index守衛純assignment；不guard掉函式呼叫。local loop bound僅接受automatic、unaddressed、nonvolatile、loop內不變、unique constant reaching definition且signed-int comparison；常數須先cast到宣告型別（unsigned char260為4），subscript equality須經CBinaryExpressionBuilder作integer promotion，否則窄索引可能錯把tracked index截斷。global/static/changing/addressed bounds不擴張。7 focused tests、qualified build/Checkstyle通過；default strict inherited警告不冒充修好。

投影前綴不能丟掉tracked-index domain：缺domain時k=200002／N=100000／i=100001／negative value仍满足prefix與exit但破壞safety。當前pure-expression contract明確拒&&／||／?:；手工reference一開始用了&&是違反既有契約，不是新resolver缺口。此案例的total scalar comparisons回0/1，可用&／|組合；不要把它泛化成會求值partial／side-effect expressions的字串替換器。

直接將VGuide嵌入ArrayAbstraction產生的CFA仍缺重建AstCfaRelation scope；最小已驗證路徑為既有C export＋normal reparse，不可放寬native scope/SSA guards假裝整合完成。短CPU export後nested ARGStatistics可能因null root在DOT visualization拋NPE；既有cpa.arg.export=false保留C export及analysis語意，fresh export exit0且與standalone export byte-identical。此export-stage UNKNOWN不當solve，之後同一scalar program的consumer對照才是解題證據。#269 STOP、#197/#215 holds、Reserved44維持。


2026-09-23 #278 補上 direct-generation 證據：使用者明確授權後，以相同實際first-spurious prompt／自動exported source，production Muse client生成3份新完整JSON+c:回答，不人工改寫；原樣消費為TRUE／UNKNOWN60CPU／TRUE，注入10／10／8且零拒收。第二份48refinements timeout，不能把全部候選接受當作必然解題，也不能忽略這份失敗。恰3HTTP starts、0重試，usage prompt4131＋completion2198＝6329tokens。新證據在reports/predicate-representation-goal-20260922/next-generation/，check_fresh.py只讀查核。

固定第一份回答的另行預登記因果對照：完整10候選兩次TRUE／1refinement；僅移除2*i<=__array_index_a、2*i==__array_index_a、2*i+1==__array_index_a這3個索引關係，其他7個不變，兩次UNKNOWN60CPU／54refinements。三個主執行加四個replay對照共62/62公式確認actual precision，皆PRECISION_ONLY並有consumer事件；replay0外部呼叫。這證明關係組在固定回答／預算下改變解題，不證明每條各自必要、跨題泛化或校準後加速。

表示路線的關鍵是讓LLM看自動array abstraction／export／reparse後的真實scalar witness source，原始回答便能直接用c:表達progress／cell-value splits，無須新quantifier resolver。原始benchmark N=100000未改；LLM的prompt表示有改，故不能把整條路線的收益只歸因成resolver微修補，也不能說舊自然語言forall回答已被自動翻譯成功。先前人工reference與本次新生成分開保存。此單案例完成必要關係→可用predicates→解題對照驗收；廣泛研究收益仍由#182承接，既有holds不解除。
