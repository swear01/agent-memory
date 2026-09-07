---
title: "參考版本失敗的成對測試不能晉升為 bug 證據"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-07
evidence_digest: 7fdeb00ccdad06f13a01682d152bc7ea6b277ff124a1bfc3dbad71218de25122
---

# 參考版本失敗的成對測試不能晉升為 bug 證據

## Problem

歷史 directed pilots 中 reference 與 buggy 都失敗，包含基準比較失敗與例外事件未被消耗而逾時。

## Durable rule

先通過 reference PASS 與目標因果事件檢查；buggy FAIL、刺激一致或保留完整檔案都不能補足失敗的參考基準。保留負面結果，禁止計入新容量。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
