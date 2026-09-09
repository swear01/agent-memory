---
title: "中斷批次應保留可驗證的已完成案例"
scope: "projects/cpachecker"
status: active
updated: 2026-09-09
evidence_digest: 32ed625b90bc74bae3b0c7fcae42199fabd31c67ceaf7f967f64ffffbfa22aeb
negative_result: false
redaction: passed
---

# 中斷批次應保留可驗證的已完成案例

## Problem

長批次中斷後只留下 incomplete XML，舊 summarizer 不支援安全逐案例合併，導致把整批已完成成果也作廢。

## Mechanism

結果完整性的判斷以整次執行為單位，沒有區分已完成案例與中斷當下的案例。

## Durable rule

保留能驗證來源與完成狀態的既有案例；重跑中斷或證據不完整的案例。逐案例合併工具尚未驗證前，不宣稱已安全續跑。

## Boundary

適用於此專案的長時間 benchmark 結果整理；不能把不完整 XML 中無法確認結束狀態的資料視為有效。

## Verification

來源記錄含使用者明確糾正整批重跑，要求修正 summarizer 並保留先前有效案例；這證明需求與錯誤決策，不證明修正已完成。

## 重開機與逐案例恢復（2026-09-09）

HAPI polling 的 `aborted` 或 `Unknown process id` 本身不能區分 host reboot、tool session 遺失或一般中止。核對實際主機的前後 `boot_id`、啟動時間、launcher／owned descendants，以及每個案例的 metadata、log 和 terminal sidecar。`boot_id` 改變能確認重開機，不能證明重開機原因；沒有保存的 launcher exit code 就保留 unknown，不猜測 signal 或成功退出。

恢復前分開盤點四種狀態：未進入 solver 的 wrapper setup failure（零 solver attempts）、可獨立驗證的 terminal case、已啟動但沒有 terminal sidecar 的 interrupted case，以及未啟動 case。依原 protocol 允許逐案例採納且來源／結果完整性可驗證時，保留 terminal case，不因其 parent wrapper 中斷而整批重跑。中斷 case 的既有檔案和雜湊原樣保留；另經具體授權的 supplemental attempt 使用新輸出路徑及遞增 attempt index，不能覆蓋 partial。未啟動 case 的第一輪仍是 attempt 1；solver attempts、terminal slots、wrapper attempts 分開計算。這不授權重試 provider 安全 halt，也不覆蓋另有整批排除要求的既定 protocol。

若接續搬到其他主機，重新核對 runtime／JDK／source／config 和資源上限，記錄 host 變更；不能把跨重開機、跨主機補跑當成乾淨的配對 timing 證據。#237 root 逐檔核對過一個零 solver setup failure，以及後續 started 2／terminal 1／interrupted 1／unstarted 4 的重開機紀錄；恢復規劃與完成結果仍是不同狀態。證據：`<experiments-root>/reports/issue237-remaining8-allocation-20260909/root-athena-reboot-receipt.json`。此記憶不宣稱接續執行已完成。
