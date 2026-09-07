---
title: "中斷批次應保留可驗證的已完成案例"
scope: "projects/cpachecker"
status: active
updated: 2026-09-07
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
