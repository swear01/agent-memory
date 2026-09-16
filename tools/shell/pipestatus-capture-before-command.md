---
title: "PIPESTATUS 必須在下一個命令改寫前保存"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: bd951e30919166e222c226dfb4883986709c28a7e8a2c4933afe3a519f71aa80
---

# PIPESTATUS 必須在下一個命令改寫前保存

保存報告指出 BenchExec 已產出全部案例，收尾卻因 pipeline_status 索引 unbound 退出；原因是先執行 local 宣告才複製 PIPESTATUS，原管線狀態已被改寫。

在執行管線前準備變數，管線結束後立即保存整個狀態陣列，再做宣告、記錄或其他命令。測試至少要含前段失敗而尾段成功的管線。

這是當時的故障回報，沒有修後執行證據；保留已完成測量不能冒稱收尾報告也已完成，負載是否影響實驗另行判斷。
