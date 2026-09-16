---
title: "包含 PredicateCPA 不代表 VGuide refiner 在執行路徑"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 23950eb57752eaacd5f291a5571d0d00f580a71d736d78b12db2b327cd52cb12
---

# 包含 PredicateCPA 不代表 VGuide refiner 在執行路徑

歷史助手先以 termination 轉 safety 推論可直接觸發 VGuide，後續讀取 construction 才指出 TerminationToSafetyAlgorithm 與 CEGAR 為分別控制的路徑，該 config 未啟用所需 predicate refiner。

沿實際 config include、algorithm wrapper 與 refiner 建立流程確認 hook 可達性，不能由 CPA 成員或功能名稱判斷。額外接上 CEGAR 是否相容及 sound 仍需獨立證据；來源只有程式閱讀與可行性計畫，沒有完成執行或證明新組合正確。
