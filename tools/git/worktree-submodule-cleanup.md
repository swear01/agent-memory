---
title: 含 submodule 的 Git worktree 非 force 清理
scope: tools/git
tool: git
tags: [git, worktree, submodule, cleanup]
status: verified
created: 2026-08-25
updated: 2026-08-25
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
