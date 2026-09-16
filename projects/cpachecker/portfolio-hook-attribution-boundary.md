---
title: "Portfolio 的泛化歸因要確認實際 deciding child"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 487021fac18d37410f8409a1e4fd70e1f27e61eae4d80e02edf04b5bed19268b
---

# Portfolio 的泛化歸因要確認實際 deciding child

歷史 config 與 Java 搜尋把 VGuide hook 限於 predicate refinement；助手後續回報 overflow smoke 有 fire 且由該 child 決定結果，並與 stock 對照區分兩邊都解與 net-new。

沿實際 portfolio routing 核對 hook、deciding engine 與 counterfactual，不能僅因載入 option 就把所有解題歸給 LLM。來源的 smoke 與後續總表是助手回報，較早 full manifest 數與表中總數也不相同；保留 run population 邊界，不外推所有 property／BAM 都受益或宣稱普遍 soundness。
