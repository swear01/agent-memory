---
title: "新 worktree 的依賴與來源解析要重新核對"
scope: tools/node
status: active
updated: 2026-09-16
evidence_digest: 60a847d69ea15284174af01c68cd8c60dfd23e6553fc353a4c1df19fed6735e8
---

# 新 worktree 的依賴與來源解析要重新核對

來源顯示 Vitest 啟動時找不到 package；歷史助手發現 worktree 沒有 node_modules，準備借用同一 base 的主 checkout 依賴。

依 lockfile、workspace 與執行環境準備依賴，再確認測試實際解析的是當前 worktree 的程式。共享 node_modules 只是當次提案，須考慮依賴狀態、安裝腳本與共用 cache；同一 base commit 不能單獨證明共享安全或隔離完整。來源沒有成功測試結果。
