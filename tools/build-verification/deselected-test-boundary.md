---
title: 測試通過數必須連同 deselected 範圍解釋
scope: tools/build-verification
status: active
updated: 2026-09-11
---

# 測試通過數必須連同 deselected 範圍解釋

一次 dataset 路徑整合後，Agent 報告 `836 passed, 4 deselected`，卻沒有說明四個測試為何被排除。使用者追問排除是否符合預期，以及移動後的路徑是否需要補測；Agent 才承諾查 pytest marker 設定、測試名稱與原因。

遇到 deselected／skipped 時，核對實際命令與設定，交代排除項目及原因，再界定通過的範圍。路徑搬移或整合後，檢查受影響的入口、replay 與資料 contract 是否已有測試；只針對確認的缺口補測，不以重跑原套件代替範圍判斷。

此紀錄證實使用者指出報告缺口；所附對話沒有後續補測結果，不能寫成「四項皆已確認可排除」或「覆蓋率已補足」。

來源：Issue25 backlog consolidation v1，controller-publication-v1 的 test-deselection 對照紀錄；正文與三則相鄰訊息已核對。
