---
title: "Help 內的 unsat 字串不能當 solver verdict"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 1939f35f095de1ef43a94337886f23e1a4cb1c1cd23b3fd9964621d9bcea5a81
---

# Help 內的 unsat 字串不能當 solver verdict

助手承認測试呼叫 pono 時用了錯誤 flag，工具印出 help，測試卻把 unsat core 子字串當成 UNSAT 成功。

先核對執行狀態與輸出格式，再解析完整 verdict；usage/help、parser error 與 solver 結果必須分開。用錯誤參數輸出的 help 做負例，不能只測正常結果。

來源沒有後續修正測試通過證據；另一次 brp2 baseline 與 guided 都逾時，不作本誤判已修復或模型有效的證明。
