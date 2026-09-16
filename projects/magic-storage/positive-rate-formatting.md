---
title: "非零產能不可顯示成零"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 6df239461d65c23b811106380e22692eb9d6168fddcae5a8521d143e88d060c1
---

# 非零產能不可顯示成零

歷史審查指出，MachineRateFormatter 固定兩位小數，MachineWorkRate.of(1, 1000) 因而顯示 0.00，讓很慢但非零的機器看似没有產能。當時測試涵蓋零及一般速率，沒有涵蓋小於顯示精度的正值。

數值格式化要保留影響判斷的語意：正速率低於顯示精度時，可顯示 <0.01 或增加有效位數。驗證零、最小正值與一般值，並核對實際 tooltip 是否使用該 formatter；不能只測 helper 的一般案例。

來源支持歷史程式審查的發現與建議，沒有本次執行或 GUI 驗證；具體格式方案仍須符合產品要求，不應把建議當成已部署結果。
