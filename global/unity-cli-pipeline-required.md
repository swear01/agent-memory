---
title: Unity 工作持續使用官方 Unity CLI 與 Unity Pipeline
scope: tools/unity; all future Unity project work
status: active
updated: 2026-10-16
---

- 使用者明確要求記住：日後 Unity 工作持續使用官方 Unity CLI ＋ Unity Pipeline（`com.unity.pipeline`）操作常駐 GUI Editor；這是跨聊天、跨專案的持續偏好，接手的 agent 也適用。
- 不以直接呼叫 Editor 的 `-batchmode`、`-executeMethod`，或冷啟動 `unity run`／`unity test` 作為替代流程。Pipeline 連線失敗先診斷與修復；確實受阻時回報具體問題，不默默退回 Batch Mode。
- C# 原始碼可正常編輯，再透過 CLI 重新編譯與 Pipeline 測試；場景、Prefab、資產操作透過 live Editor 執行。驗證包含實際 Play、影格與控制器時間前進、輸入、畫面擷取及 Console。
- Gamejam1 已將此規則寫入專案 AGENTS.md 與 Development.md；本次已實際用 CLI ＋ Pipeline 完成場景建立、17 項 Edit Mode／1 項 Play Mode 測試及 live 輸入驗證。不能把僅啟動 Editor 或送出命令當作驗證完成。
