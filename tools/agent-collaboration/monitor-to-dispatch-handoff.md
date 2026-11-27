---
title: "監控接續前核對完成通知的 run 身分"
scope: "tools/agent-collaboration"
status: active
updated: 2026-09-08
evidence_digest: 95ceb4ec47df68ac04d9ae6de4210068d005ecb073fdb8af1b4a0d768fa15c0b
---

# 監控排程必須驗證下一批接續

先核對完成通知對應本次 run ID、start time、程序身分與終結產物。歷史案例收到的是上一趟緩衝摘要，當下程序與輸出仍顯示正在執行。PID 或 elapsed 只輔助找範圍。來源沒有監控器修補證據。

分批 benchmark 或背景佇列要驗上一批終結、產物、容量判斷與下一批實際啟動。沒有容量時保留等待原因及續查時間；timer 存活或測試綠燈不能證明真的曾接續。歷史案例只監控、沒派下一批，之後 scheduler 測試通過，但當時仍無可用容量供實際交接驗證。

[接續案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/agent-collaboration/monitor-to-dispatch-handoff.md) · [通知誤判案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/agent-collaboration/buffered-completion-identity.md)
