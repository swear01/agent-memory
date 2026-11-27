---
title: "Agent 工作樹稽核應處理巢狀結構與孤立父儲存庫"
scope: "tools/git"
status: active
updated: 2026-09-16
---

## 巢狀、孤立與有未保存工作的工作樹

- 頂層沒有 `.git` 不代表不是 task 根目錄。檢查巢狀 repo 的 `.git` file／directory，以及 `git worktree list` 和 session 實際 cwd，避免把真正的工作樹誤作 `not_a_worktree`。
- `fatal: not a git repository` 可能是 worktree 的 `.git` 仍指向已搬走或刪除的主 repo。讀 `gitdir:` 確認；無法用主 repo 的 `git worktree remove` 時，仍須先盤點未保存資料與工作歸屬。
- Dirty、仍在使用或無法驗證的工作樹保留。Unity `Library/`、`Builds/` 與 .NET `bin/`、`obj/` 常占大部分空間，但這不代表工作樹可刪。分開核對 tracked、untracked 和可再生產物，能確認歸屬及再生方式的產物才按已授權範圍清理。
- 這筆歷史只支持稽核與分類方法，不能把匯出 patch 當成刪除 dirty worktree 的授權。原文見 `git show 2505b5e:tools/git/worktree-audit-nested-and-orphan-cleanup.md`。

歷史驗證中的 worktree 曾被移除，分支仍在，但未提交修改已遺失。清理前先確認沒有使用中的驗證、狀態乾淨，並讀回可復原證據。當時只能重做，來源沒有重建成功證據。見 `git show 2505b5e:tools/git/preserve-uncommitted-verification-worktree.md`。
