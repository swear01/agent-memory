---
title: 原生 C 謂詞接入的純度、scope 與 trace context 邊界
scope: projects/cpachecker
status: verified
updated: 2026-09-21
---

CPAchecker 原生 CParser／PathFormulaManager 可以重用，但一般 statement fragment helper 不等於純 expression parser：x++ 可被 ASTConverter 降成暫存 expression，副作用留在 Sideassignments。外部候選須先檢查 CDT 原始 AST，再確認沒有 pre/post/conditional side assignments；empty include provider 加上 preprocessing 拒絕可防止 fragment 讀取 include 檔案。既有 witness helper 不應為此改變語意。

AstCfaRelation 的 active-declaration snapshot 不包含 nearest-shadowing 順序，同名 outer/inner locals 可能並存。以原始名稱找候選，unique local 優先 global，再比對 parser 的實際 declaration；多個同名 local、已離開作用域、foreign 或缺少 metadata 都明確拒絕。CProgramScope 的寬鬆 fallback 不能代替這個檢查；native C encoder 只在 C CFA 初始化，避免破壞其他 frontend 的既有 SMT 路徑。

使用與 block formula 相同的最後 aligned trace occurrence 的完整 PathFormula（SSA 與 pointer targets）。最後一次 context 缺失時不可退回較早 SSA。若編碼引入新的 SSA／pointer-target state，先報 unsupported context。顯式 c: 標記失敗不得改用其他 grammar 猜測。預期 parse／transfer／argument 錯誤可拒絕；意外程式缺陷不可被 broad RuntimeException catch 偽裝成候選品質問題。

PR268 已合併；初始 84 tests，加上最終受影響 31 tests，共 87 個不同 focused tests 通過。真實 CFA 編碼、型別／signedness、scope、同一 context 與 predicate/local precision 邊界有回歸檢查。一次 exposed-source consumer probe 顯示原 mixed-SMT 字串被拒、人工 c: 關係可插入 precision；其構造 prefix 未驗證可行性，不是模型生成、adequate oracle 或 solve-gain 證據。native-specific rejection 尚不在既有付費 repair allowlist；改變生成語法和 repair-call policy 必須分開歸因。

實際 FIRST_SPURIOUS 消費檢查（2026-09-21，既有 exposed sorting case）已超過上述構造 prefix：相同新 runtime／首次 preselection context 下，保留原9個有效 candidate objects 有10個 head bindings；人工加入原生相鄰關係後有11個，皆為 PRECISION_ONLY。新增關係在實際 N45／SSA／heap context 編碼，並在11次有界診斷中到達 definitionPush、AllSAT 和正常返回；10次有 callback。兩邊仍為600 CPU秒 timeout、各93 refinements，沒有新解題或模型生成證據。診斷預算用盡後的缺少事件不能當作沒有使用。

原生 C 編碼成功不補足數學證明義務：單點 a[x] <= a[x+1] 不等於有界量化的全域排序／跨迴圈關係，head／exit 的 C 存取還需範圍條件。另一方面，PRECISION_ONLY 是抽象分割特徵而非程式假設，不能只因候選非 invariant 就宣稱 verifier 不 sound。換 prompt 後請求 hash 會變；人工搬移歷史 response 必須明說是 intervention，先捕獲新的 production request，再驗證 context／cache 身分，不能冒充新的模型回答或原請求的 cache hit。研究紀錄：reports/issue259-native-refinement-20260921/，父 issue259／103 保持開啟。

另一個已知 mechanism case 的新生成結果不能與上述 sorting 負例混為一談：#151 在同一 8c3c822 runtime 的 cggmp2005_variant 上，先捕獲 actual FIRST_SPURIOUS prompt，確認 contract 有 main 的 lo／mid／hi，再以 Muse 做 control／affine cue 各3次。六份完整新回答皆含 hi = lo + 2*mid 的等價 c: 表達式，原樣經 production validation／PRECISION_ONLY injection 後皆 TRUE（五份1次 refinement，一份2次）；empty response control 在300CPU timeout，既有 affine+bound reference 為 TRUE。這是 native C 路徑可承接模型實際有用輸出的單案例證據，不是新 hard218 gain；兩組3/3平手，不支持 cue 優越性。模型／runtime／prompt 都不同於舊 DeepSeek interrupted cohort，不能歸因為單一 scope 修補或填回舊缺失 draws。外部 cue 生成的 response 對共用 consumer request 的映射須明確保存，不能假稱 treatment prompt 本來就是 runtime 產生。研究紀錄：reports/issue151-fresh-generation-20260921/；單一謂詞族的必要性須另看移除對照，不能從完整回答 TRUE 自動推定。


2026-09-21 的 #271 全量轉換調查在 main `64d5888775a9d99c35c2a26a6dbb4aaccff9da8a` 確認上述保守 shadowing guard 的實際能力缺口：zero_sum4 的 N29 同時保留 outer main::i／inner main::i__1，CFA condition 使用 inner，但 function-wide parser 把 bare i 解成 outer；按 origName 計數又使精確 alias i__1 也被拒。167 份原生 C 回答中共24筆相關拒收（8 bare i、16 alias）。N35 離開 inner 後 outer 可用、inner 正確拒收。最小下一步是 head lexical declaration identity，不能刪 guard、以最大 source line 猜 nearest scope，或只改 prompt 名稱。這是尚未修補的表示能力缺口，不是新增解題。

同次調查把所有接受公式逐一 uninstantiate 後，與保留的實際 post-injection local precision 比較：1,411/1,411 exact members；221筆沒有獨立結果的 head/raw pairs 全為 whitespace／canonical formula aliases。這不證明各公式有後續 solver 消費或解題因果。7個原生 C 控制例（promotion、signedness、array point、2D、pointer、remainder、bitwise）與 CFA edge 等價，支持重用已有 encoder。synthetic SSA fixture 的 CType 要依原生 pathformula 慣例移除 scalar qualifiers；const CType 失敗不能誤判為真實模型轉換缺陷。

現成 ArrayAbstraction 不能假定與 VGuide native C plug-compatible：原始 array_init_pair_symmetr2 有3 eligible arrays／0 eligible loops，回 UNCHANGED；TransformableLoop 以空 ValueAnalysisState 找 constant comparison，無法直接承接 i < N。簡單 for／while 控制例各轉1array／1loop、PRECISE，但轉換後全部23／22 nodes 查不到 AstCfaRelation scope；CFA transformer 沿用舊 metadata，source prompt 仍讀原 source。需要另做 eligibility、scope／source context 與 witness 表示資格檢查，不能把改 JSON／開 config 當作量化關係已接通。研究紀錄：reports/conversion-gap-audit-20260921/，issue271；沒有 verifier／provider run。
