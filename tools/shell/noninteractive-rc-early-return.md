---
title: "Source shell rc 仍可能被非互動 early return 擋住"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 46582dc9b2f7da18988b44d9c67534981245a13d26b5b560aa76d151f74b77cd
---

# Source shell rc 仍可能被非互動 early return 擋住

歷史助手先建議 batch 啟動前 source shell rc，後查到檔案前段遇非互動 shell 就 return，因此位於後段的 key 設定根本沒有執行。

確認真正的啟動模式與 rc 控制流程。批次需要的設定應透過既有安全機制明確提供，不能假設 source 就會載入所有內容；檢查只確認存在性，不印密鑰。互動 shell 是當時提出的做法，不是所有自動化的通用修法。來源沒有批次恢復成功的結果。
