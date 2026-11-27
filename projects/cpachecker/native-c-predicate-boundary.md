---
title: 原生 C 謂詞接入的純度、scope 與 trace context 邊界
scope: projects/cpachecker
status: verified
updated: 2026-09-23
---

- `c:` 外部候選在 CDT 原始 AST 檢查純度，再拒 pre/post/conditional Sideassignments；`x++` 可被 ASTConverter 降成純-looking temp。Empty include provider 搭配 preprocessing 拒絕，保留 witness helper 語意。顯式 `c:` 不能退回猜其它 grammar，意外缺陷不要 broad RuntimeException 吞成拒收。
- C scope 使用 lexical name→declaration snapshot 經 ParseResult／AstCfaRelation 給 `CProgramScope.withVariableBindings`。離開 compound、for、statement expression 時，在 `leaveBlock(successor)` 後刷新可重用的 successor CFANode。以原 source spelling 優先於可用 generated alias，拒 foreign、過期或缺 metadata 的 binding，勿猜 source line 或用 function-wide lookup。只在 C CFA 啟用。Real-CFA 控制不受 synthetic SSA fixture 的 scalar qualifier 問題誤導。
- 使用與 block formula 相同、最後 aligned occurrence 的完整 PathFormula（SSA／pointer targets），缺失不回退，拒新增狀態。Native-only batch 可跳過 legacy array template，mixed 保留。用 uninstantiate 後的 exact membership 讀 actual precision；接受、注入、consumer reach、refinement 次數和 solve 各自核對。PRECISION_ONLY 是抽象分割；全路徑不變量需另證。
- ArrayAbstraction 直接轉換會漏 AstCfaRelation scope，已驗證路徑用 C export + normal reparse。若 nested ARGStatistics 輸出 DOT 遇 null-root NPE，可設 `cpa.arg.export=false` 並核對 C export 不變。Export-stage UNKNOWN 另記。轉換前查 eligible loops；以空 ValueAnalysisState 推常數會漏局部 loop bound。
- 多 subscript 初始化要保留迭代與每次寫入，以 tracked index 守衛純 assignment，不得 guard 掉 calls。局部 bound 限 automatic、unaddressed、nonvolatile、loop invariant、唯一常數 reaching definition 及 signed-int comparison；常數 cast 到宣告型別（如 unsigned char 260→4），subscript equality 用 CBinaryExpressionBuilder 的 integer promotion。投影須保留 tracked-index domain。純表達式拒 `&&`／`||`／`?:`，total scalar comparisons 才可用 `&`／`|` 組合。
- 分開人工 prefix/reference 與新生成回答。#278 在自動 export/reparse 的 scalar source 上有 direct-generation 一題 TRUE/UNKNOWN/TRUE，固定回答只移除三個 index relations 時由 TRUE 轉 timeout。這支持該組關係的單案例因果，未證每條必要、跨題或速度。#271 shadowing 修復在 fixed replay 增加有效 precision、減少 refinements，兩臂仍 timeout。切 prompt hash 先捕獲新 production request，人工借舊回答記 intervention。研究入口 `reports/predicate-representation-goal-20260922/` 與 #182。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/native-c-predicate-boundary.md)
