---
title: 含 submodule 的 Git worktree 非 force 清理
scope: tools/git
tool: git
tags: [git, worktree, submodule, cleanup]
status: verified
created: 2026-08-25
updated: 2026-09-07
---

# 症狀

乾淨的 task worktree 曾在執行普通 `git worktree remove <task-worktree>` 時失敗：

```text
fatal: working trees containing submodules cannot be moved or removed
```

即使先執行 `git submodule deinit -f -- <submodule>`、submodule working directory
已清空，某些 Git 版本仍會因
`<common-git-dir>/worktrees/<worktree-name>/modules` 的 cached repository 而拒絕。

# 已驗證的非 force 流程

1. 確認 superproject 與每個 submodule 都是 clean，並確認 exact worktree ownership。
2. 在 task worktree 執行 `git submodule deinit -f -- <submodule>`。
3. 把該 worktree metadata 下的 `modules` cache 搬到可回復、唯一命名的暫存目錄。
4. 執行普通 `git worktree remove <task-worktree>`；若失敗，立刻把 cache 搬回原位。
5. merge 後先讓乾淨的 local base checkout `git merge --ff-only <remote-base>`，再從
   該 base worktree 執行普通 `git branch -d <task-branch>`。
6. worktree 與 branch 都確認移除後，把暫存 cache 搬入使用者 Trash 保存。

整個流程不需要 `git worktree remove --force`、`git branch -D` 或 `rm -rf`。
cache 是 deinitialized submodule repository，必要時也能從 remote 重新初始化；
若 ownership、cleanliness 或 cache path 無法精確驗證，保留 worktree 並停止。

## 新建 sparse worktree 的 empty index

`git worktree add --no-checkout` 後直接 `git sparse-checkout set --cone ...`，
本次新 worktree 的 index 仍為空，`git status` 因而列出整棵 tree 的 staged
deletions。這不是可忽略的正常 sparse 狀態。僅在確認是自己剛建立、尚無
任何使用者或 worker 修改的 worktree 後，執行 `git read-tree -mu HEAD`
初始化 index／checkout，再確認 porcelain 為空才派工。既有或來源不明的
dirty worktree 不可套用這個修復；先保留工作並診斷。
