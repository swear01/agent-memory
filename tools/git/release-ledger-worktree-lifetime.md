---
title: "Release ledger 必須跨過 worktree 清理存活"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: ddf329d032e0e8f766305cd9fb5760dc382f64f4554a3fcd2a1cf3530129928f
---

# Release ledger 必須跨過 worktree 清理存活

歷史助手發現 release 缺 manifest/audit，manifest 曾隨 worktree 刪除而遺失；從 patches 重建時又指出 resolution 使 reverse-check 不可靠。

刪除工作樹前確認交付所需 ledger 已保存到指定耐久位置並可讀回。需要重建時依實際決策與最終內容核對，不能以套回 patch 是否成功替代完整來源。來源沒有重建後驗證結果。
