---
title: gh pr merge --delete-branch 不能刪掉仍被 worktree checkout 的本機分支
scope: tools/git
tool: gh
status: active
created: 2026-09-14
updated: 2026-09-14
tags: [git, worktree, gh, merge]
---

# `gh pr merge --delete-branch` 與 agent worktree

在 task worktree 裡執行：

```bash
gh pr merge <n> --merge --match-head-commit <sha> --delete-branch
```

可能先失敗：`'main' is already used by worktree at <main-checkout>`。
改到 main checkout 再執行。GitHub 端仍可能已經合併成功；先 `gh pr view`
確認 `state=MERGED`，不要重送 merge。

遠端分支刪掉之後，本機 `git branch -d <feature>` 仍會失敗，若另一個
worktree 還 checkout 該分支：

```text
cannot delete branch '<feature>' used by worktree at <task-worktree>
```

正確順序：確認 GitHub 已合併 → 確認 task worktree 乾淨 → 普通
`git worktree remove` + `git worktree prune` → 在 main checkout
`git branch -d <feature>`。不要為了刪分支對 worktree 用 `--force`。

2026-09-12 在 `deepseek-latch-gateway` PR #14 與 docs PR #15 各發生一次，
GitHub merge 都已成功，只有本機刪分支失敗。
