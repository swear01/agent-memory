---
title: "Issue25 完成關卡後必須退役自動啟動鏈"
scope: projects/issue25-memory-promotion
status: active
updated: 2026-09-09
---

# Issue25 完成關卡後必須退役自動啟動鏈

## Incident

Athena 重開機後，已完成的舊 v5 建置、分片驗證與 logical-parity 工作又被排入或啟動。新訊息審查停在保存進度，熱保護則因 `After=default.target` 等待舊 oneshot 建置而尚未啟動。

這些 unit 來自早期 pilot → full build → shard validation → assembly → G0–G16 自動接續設計。保存的檔案修改時間落在早期部署期間；這只能支持來源與年代，不能單憑 mtime 指認是哪一位控制者最後 enable。切換到分批訊息流程後，舊啟動入口未一併退役；接手與先前恢復時只看執行中的新服務，漏查 enabled units、path triggers 與 pending jobs。

## Durable rule

- 一次性關卡完成或流程換版時，盤點其 enabled service、target、path、timer 與反向相依。保留報告和 checkpoint，停用已退役入口；不要只停止當下 PID。
- `ConditionPathExists` 只限制該 unit 是否執行，不能當成整條相依鏈不會啟動的保證。此例 G8 沒有執行，舊前置工作仍被拉起。核對 Wants/Requires/After，而不只核對完成檔案。
- 熱保護應能獨立於耗時計算啟動。此例改為 `Before=default.target`，現行審查及下一組初篩加入對 guard 的 Requires/After；正常 CPU 配額仍為 80%，緊急降載與凍結保留。
- 重開機恢復時先查 boot ID、pending jobs、enabled units、實際 PID 與熱保護，再驗證保存輸出後續跑。stale running 進度檔不能證明服務存活。
- 取消相依 job 可能連帶移除其他 job；遇到 job ID 已不存在，重新列出實際狀態，不能因批次 cancel 非零退出便略過後續精確停止與驗證。

## Verification

本次實際保存設定後停用 34 個過時入口，停止冗餘建置及 34 個舊 validation/logical-parity workers。恢復後只有 guard 與現行來源審查執行，沒有 pending jobs，只有 guard 保持 enabled。G8 本次 boot 的啟動時間為空；未將其重新執行。保存的 179 個檔案、110 批輸入、先前帳冊及 41 篇 canonical 筆記雜湊均通過核對；來源審查從 70/110 接回。

後續 guard 排序與 worker 相依修正經 daemon-reload 後，以 effective properties 驗證，live PIDs 不變。未刻意重開機測試；未確認造成主機重開機的根因。這是啟動生命週期錯誤，不能據此宣稱主機重開機由 Issue25 或過熱造成。

## Reference

[systemd unit dependency and condition semantics](https://github.com/systemd/systemd/blob/main/man/systemd.unit.xml)
