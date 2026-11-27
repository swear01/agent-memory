---
title: HAPI Session Status 面板的唯讀與獨立區塊
scope: tools/hapi
status: needs-review
updated: 2026-11-28
---

使用者對 HAPI Session Status 面板的需求：唯讀呈現 Goal、Task 清單、Subagents 和 Background terminals，各有明確來源和狀態。Task 用最新 snapshot，不累積舊清單，也不猜 Task 和 Subagent 屬於同一階層。錯誤放回所屬項目，無錯誤或空區段時無須填充；額度資訊不混進面板。

背景 terminal 應可辨識與查看；依該要求不加 Stop、Re-run 等更動執行的按鈕。若需導航查看輸出，與程序控制交互分開。這些是規格討論，不表示 backend 事件格式、區塊或端到端流程已實作。驗收以實際事件與現行畫面資料為準。

[原本唯讀要求](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/status-panel-readonly-scope.md) · [原本區塊要求](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/status-panel-independent-sections.md)
