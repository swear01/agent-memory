---
title: "Worktree typecheck 要確認 workspace package 真實路徑"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 1815263897a5d3e4d90948ee53868b5f59919ff47b8d34fc2203697be01c06b7
---

# Worktree typecheck 要確認 workspace package 真實路徑

使用者 hub typecheck 缺 serviceTier 與新 schema，CLI 同時沒有錯；助手診斷 worktree node_modules 透過主 checkout 的 workspace symlink 讀到舊 shared protocol。

先查實際 resolver／symlink 指向與套件版本，再修改型別或安裝依賴；worktree 目錄名稱不保證所有 consumer 都用該分支。來源有錯誤輸出與診斷，未提供修正後 typecheck，不應把暫時連結調整稱為 runtime 功能已完成。
