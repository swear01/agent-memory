---
title: "Exception 字串計數不能當 crash 數量"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 6e7251bb5176ead27254c84fa48c2383790242f8e2780db67e6e7319b97fc33b
---

# Exception 字串計數不能當 crash 數量

保存的兩個 recursion smoke rows 將 CRASH 記為 2，但原始命中行是 ExceptionHandlingAlgorithm 的 handled WARNING；後續 log 有 UNKNOWN incomplete analysis，uncaught／severe 計數為零。

依退出狀態、未捕捉例外、最終 verdict 與子分析結果分類，不以字串命中直接標崩潰。來源能更正這兩個計數，不能證明所有 runtime 都無故障；stock 那列缺最終 verdict，更不能據此作因果比較或把小 smoke 稱為完整泛化驗證。
