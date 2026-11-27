---
title: "Issue25 完成關卡後必須退役自動啟動鏈"
scope: projects/issue25-memory-promotion
status: active
updated: 2026-09-09
---

# Issue25 systemd 舊入口退役與熱保護

- 工作完成或換流程時，盤點 user service、target、path、timer、enabled 入口和反向相依。留報告及 checkpoint，停用退役入口。舊 PID 結束或 `ConditionPathExists` 跳過單一 unit 仍可能拉起前置工作，讀 `Wants`／`Requires`／`After` 與 pending jobs。
- guard 要能比耗時計算早跑，該次以 `Before=default.target`，現行 workers 帶 `Requires`／`After`。Worker 要符合 guard 的選取範圍（舊 `issue25-v5-g*.service`），在同一 user manager 核 effective properties、PID 和 guard log Active Worker。System 範圍 not-found 不表示 user unit 已停。
- 保留 `AllowedCPUs` 和 80% 正常 CPU 配額，動態熱保護及緊急凍結優先。執行 `daemon-reload` 後查 effective properties 與實際 worker；該次沒有刻意加熱或 reboot。
- 重開機後先讀前後 boot ID、pending jobs、enabled units、熱保護及現存 PID，再驗證保存輸出。取消相依 job 可能一併移除其它 ID，重新列出實際狀態再操作。stale 進度檔不能證明服務活著。啟動鏈錯序及 worker 選取錯誤各有不同證據，重開機根因未確認。

[systemd 官方 unit semantics](https://github.com/systemd/systemd/blob/main/man/systemd.unit.xml)

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/issue25-memory-promotion/autostart-retirement.md)

[合併原文](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/issue25-memory-promotion/thermal-guard-worker-selection.md)
