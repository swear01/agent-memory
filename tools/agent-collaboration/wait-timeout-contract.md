---
title: "等待工具的最小 timeout 是輸入契約"
scope: "tools/agent-collaboration"
status: active
updated: 2026-09-07
evidence_digest: b7793add8b1cc942d4594e58d80ed0603bfaf41bb5572f92ea7193fedf07ebb6
negative_result: true
redaction: passed
---

# 等待工具的最小 timeout 是輸入契約

## Problem

wait_agent 的過短 timeout_ms 被驗證器拒絕，第一次呼叫沒有真正開始等待。

## Mechanism

呼叫端給的等待時間小於當時工具的 10000 ms 最小值。

## Durable rule

依有效工具 schema 的下限設定 timeout_ms；輸入驗證錯誤不代表工作完成或等待逾時。

## Boundary

來源當時的下限是 10000 ms；未來工具版本以當次提供的 schema 為準。

## Verification

來源工具回覆明確指出 timeout_ms must be at least 10000，接著記錄了使用 10000 的重試；不把重試動作本身當成下游任務完成。

## Negative result

This memory preserves a verified negative result.
