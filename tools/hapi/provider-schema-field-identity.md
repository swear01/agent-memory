---
title: "Provider 欄位映射以實際協定 schema 為準"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: e01ab43033e10e0c033d0457a6f807fd61b56a9a77fea05625e7de73cc2396a6
---

# Provider 欄位映射以實際協定 schema 為準

歷史助手檢查 Codex 0.145.0 時回報 rate-limit schema 使用 windowDurationMins 與 resetsAt，但 HAPI normalizer 尋找 windowMinutes、reset_at 或 resetAt，造成欄位不匹配。

整合外部事件時對照目標版本的真實 schema 與 payload，逐欄位驗證名稱、型別與缺值處理；不要把猜測的命名當成協定。來源是歷史查核自述，沒有修補後結果，也未確認更舊版本是否使用其他名稱。
