---
title: HAPI storageInsights 的 dbstat 可用性與估算邊界
scope: tools/hapi
status: active
updated: 2026-09-11
---

# HAPI storageInsights 的 dbstat 可用性與估算邊界

歷史 Agent 報告記載：storage insights 在 Linux CI 回傳 GET 500，追查為當時 Linux Bun 1.3.14 build 缺少 `ENABLE_DBSTAT_VTAB`，而開發用 macOS build 有此功能。原文說 production hub 也是 Linux，因此有部署風險；它沒有證明正式環境已發生故障。

原報告記載的修補是 `storageInsights()` 在 dbstat 缺失時改以欄位內容長度估算每表 bytes，另保留精確行數，並以 `breakdownApproximate` 讓 API／UI 明示估算。測試包含 fallback 路徑與 nullable 欄位加總問題。這是歷史修補及 CI 通過的轉述，本次沒有重跑該 HAPI 版本。

可重用的檢查方式：對依賴 SQLite build 選項的功能，確認目標 runtime 實際支援情況，並測試缺少功能的路徑。內容長度估算不能冒充 dbstat 的頁面配置／實體磁碟用量；正式環境驗證與 Linux CI 結果分開記錄。未來 Bun build 是否仍缺少 dbstat，須重新檢查。

來源：Issue25 backlog consolidation v1，controller-publication-v1 的 dbstat-ci-scope 對照紀錄；正文與四則相鄰訊息已核對。
