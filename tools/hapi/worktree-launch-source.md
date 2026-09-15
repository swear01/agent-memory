---
title: "測試啟動腳本必須使用修改所在的 worktree"
scope: "tools/hapi"
status: active
updated: 2026-09-15
evidence_digest: 00698bbcfa6e2d0f1bbe868b0e82498b2fc87b8c01a789342f5e1980d976cf9a
---

# 測試啟動腳本必須使用修改所在的 worktree

使用者回報看不到剛修改的介面。歷史來源中 Agent 確認 quick restart script 的 cd 與 workspace-root 都硬寫 main checkout，而修改位於另一個 worktree，因此先前啟動的是舊版本。後續摘要回報改用 worktree 重建並觀察到新的 asset hash。

測試前先讀 .git/info 中的 local notes，再核對啟動腳本的工作目錄、workspace-root、實際 process argv 與 served artifact。改動位於 worktree，不代表 restart script 會自動使用它；驗證具體功能與載入版本後再判斷瀏覽器 cache 是否相關。

來源直接支持啟錯 checkout 與使用者糾正；新的 asset hash 是歷史摘要中的回報，不等於本次已驗證 UI 或 cache 是剩餘問題的原因。
