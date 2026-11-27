---
title: Cursor CLI 個人方案的剩餘配額來源
scope: tools/cursor
status: active
updated: 2026-11-28
---

查 Cursor 個人方案配額時，CLI 的 status、about、模型 context 使用量和團隊用量事件都不足以驗證剩餘比例。2026.11.22-e9042b2 的原生用量面板使用 DashboardService.GetCurrentPeriodUsage。以既有 Cursor 登入呼叫該唯讀服務，可以驗證 planUsage 內哪些百分比及 billingCycleEnd 確實存在。它是 CLI 使用的介面，供應商可能改版；服務回覆缺少一個用量池時就省略該池，空的回覆不能推算成剩餘 100%。驗證只需檢查欄位與數值範圍，毋須輸出憑證或個人數字。

[實作與測試](https://github.com/tiann/hapi/pull/2631) · [官方各方案用量入口](https://cursor.com/help/models-and-usage/usage-limits)
