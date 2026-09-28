---
title: "LLM predicate 的回饋缺口與不變量學習邊界"
scope: "projects/cpachecker"
status: active
updated: 2026-09-28
---

在凍結 revision `e8ffb7aace30d6c2ffc323da3e38ee9fbd15bb7b` 的 native75 中，LLM 前端已有多輪 schedule、最近4輪 CE／outcome、precision 累積與 source slicing。重新提議這些功能之前，先辨認真正缺少的資料。

本次只讀稽核151個完成呼叫：全部structured CE明列branch_conditions／ssa_values／assignments unavailable，全部relation summary含省略號；CeSummaryBuilder將單條relation截為280字元。原source仍可能含分支與賦值，所以不能說模型完全看不到它們。structured assertion雖空，但151個prompt均正確明示G ! overflow，不是active property缺失。

RefinementOutcomeStore只有visits、interpolant/block數、候選接受／注入／拒收數與native_delta；最後一項在LLM注入前計算，不能當模型效益。EVERY有111個後續prompt明列infeasible pivot、ARG prune等資訊unavailable。PredicateUsefulnessGate在所有稽核record均disabled，不能拿其歷史bvmul heuristic解釋這批結果。

EVERY的1,506 candidate items展成1,683 head bindings；以同題同arm的head＋僅規範化空白的raw文字計算，795次重複出現（47.2%）。這不是solver語意等價、不表示47.2%可省成本，也不能忽略相同head在不同occurrence／memory context的語意差異。接受／注入數、唯一公式數、局部進度與原題verdict必須分開。

Clause2Inv／Loopy的clauses與Houdini有可借用的候選保存及具體失敗回饋，但不能把PRECISION_ONLY split一律當待證inductive invariant篩掉。ICE正例、反例、implication labels須由相應teacher產生；假反例整條path不可滿足，不能捏造它的concrete model。局部SAT prefix／transition model須標明可能不可達。沿實際ARGPath列完整CFA edges，是較小、尚待效果測試的context補充方向；勿另造parser或擅自啟動已退休precision compiler。

舊Issue152已明確RETIRED／NOT_PLANNED，單例obligation提示／factorization未證泛化。不可重跑舊停止格，再把同題成功稱為通用；新研究需新範圍、跨題型、answer-free與等長無關資訊control。先把context與scheduler分開消融，維持原source、機器語意、consumer及whole CPU/wall。

上述歷史只讀稽核沒有新 verifier／模型呼叫，未產生新增解題或加速結論。可追溯報告與雜湊：
`<experiments-root>/reports/predicate-generation-methods-20260928/REPORT.md`、
`EXISTING-RUN-AUDIT.json`。


#296 後續實測已確認實際 prompt 收到完整 CFA proof steps（第一對相同 prefix：0筆對56筆，16,000字元上限），但資訊抵達模型不等於 predicate 可消費或新增解題。首輪 direct CEGAR pilot 漏掉 `cpa.predicate.memoryAllocationsAlwaysSucceed=true`：凍結2026-07-11 corpus 的 `sll-token-2` 因允許 malloc 回傳0而三組同判 WRONG。只切換此選項的無 LLM 對照消除空指標反例，但 MathSAT5 插值仍失敗，結果 UNKNOWN。從既有 reducer 配置拆出 direct baseline 時，必須保留原 corpus 的共同 C 語意契約，同時依每題 YAML 保留 ILP32／LP64；不能整份複製 reducer 選項，也不能把共同語意一起刪掉。證據：`<experiments-root>/reports/issue296-allocation-semantics-20260928/`。

#298 確認另一個轉換缺口：`parsePureExpression` 的無宣告 wrapper 讓 CDT 將 `~state_145` 設為問題型別；後續 scope binding 修正 identifier，`ASTConverter` 的 unary default 卻保留舊 `CProblemType`，導致公式編碼拋 `UnsupportedOperationException`。候選是有效的 C 運算；只新增拒收 guard 可止崩潰，但不算修好它的表示能力。修補需從已解析 operand 推導 unary 型別，另保留對未解析型別的個別拒收。直接 exit1 的未處理例外須另記 INVALID／failure reason，不應當成普通 budget UNKNOWN 或負面方法證據。PR #299 已合併修復：僅對 `op_tilde` 從有效整數 operand 套用 C integer promotion，保留 unary minus 原行為；另於 native encoder 遞迴拒收 CProblemType。原題使用同樣兩份快取回答、零新 API 呼叫重播，原崩潰候選成功在 N582 注入；第二輪15個全部接受。原題仍到診斷CPU上限，故 consumer修復不等於新增解題。證據：`<experiments-root>/reports/issue298-original-replay-20260928/CONSUMER-EVIDENCE.json`。


#300 記錄另一個實際入口限制（來源479804e）：`PredicateCPARefiner` 先 `checkCounterexample` 再進 VGuide bridge；原生插值失敗可在任何 LLM 呼叫前結束分析。`fallbackWithoutInterpolation` 即使 non-interpolating solver 證明 UNSAT 仍重拋原插值錯誤；INFO log 不能區分此分支與 feasibility check 也失敗。不可因看見插值錯誤就宣稱路徑已證 UNSAT。現成 `sourcePriorMode=true` 只接受SMT路徑，`validateSourceOnly` 明確以 `native_c_context_unavailable` 拒收所有 `c:` 候選；它缺乏實際 block formula／SSA／pointer context，不是 native-C 的可直接開啟替代路線。研究提前介入時，先核對真實 context 是否已存在，再重用現成 encoder／validation，不能猜 SSA 或偽造 interpolants。證據：`<experiments-root>/reports/issue296-interpolation-boundary-20260928/EVIDENCE.{md,json}`。
