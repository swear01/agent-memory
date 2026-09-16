---
title: "幂等重入檢查的 marker 內容也要符合契約"
scope: tools/experiments
status: active
updated: 2026-09-16
evidence_digest: 71d2714d771ac82876da5f7f4be784f9af22e1eddc94e958afabe9abdbf19408
---

# 幂等重入檢查的 marker 內容也要符合契約

歷史 runner 的 completed-output 路徑用 touch 建立空 marker，但已驗證 summary 中的 marker 包含 complete 換行，遞迴 diff 因此失敗並退出1。

對照完成檢查器預期的完整內容，包括 marker bytes；不能只確認檔案存在。修正驗證端時保留既有已完成輸出，另測重入成功與目標不變。來源報告前後 target hashes 相同，沒有後续修補結果，不能宣稱已完成恢復。
