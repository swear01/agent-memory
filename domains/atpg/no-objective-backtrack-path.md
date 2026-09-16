---
title: "迭代數增加不代表搜尋已脫離無進展迴圈"
scope: domains/atpg
status: active
updated: 2026-09-16
evidence_digest: 4c0be22d8ca514fd3bcc6308a0fcfcbf90169a2a7749f211f6e477e65150806e
---

# 迭代數增加不代表搜尋已脫離無進展迴圈

來源顯示 exit124，debug 迭代數持續增加但 gate 與 backtrack 值不變；助手承認先前修法只是把無限迴圈移到更高層，準備檢查所有 findFinalObjective 呼叫。

找不到 objective 時核對共同控制流程是否確實回溯或終止，並驗證所有呼叫路徑。進度計數只能證明執行經過該處，不能代表收斂；來源沒有修正後求解或測試結果。
