---
title: 含 submodule 的 Git worktree 清理
scope: tools/git
tool: git
tags: [git, worktree, submodule, cleanup]
status: verified
created: 2026-08-25
updated: 2026-09-28
---

# 症狀

乾淨的 task worktree 曾在執行普通 `git worktree remove <task-worktree>` 時失敗：

```text
fatal: working trees containing submodules cannot be moved or removed
```

即使先執行 `git submodule deinit -f -- <submodule>`、submodule working directory
已清空，某些 Git 版本仍會因
`<common-git-dir>/worktrees/<worktree-name>/modules` 的 cached repository 而拒絕。

# 使用者核准的直接清理

2026-09-28 使用者要求刪除 `personal-pr-workflow` 中禁止強制清理的敘述，
並清掉已合併的暫存 worktree。shared-skills PR #42 已移除該句；保留
ownership、cleanliness 與 inactive 檢查。不要再把 `--force` 本身當成禁止事項。

確認 task commit 已在遠端合併、worktree 沒有未保存內容、submodule 沒有
未推送工作且沒有程序使用後，可從主 checkout 執行：

```bash
git worktree remove --force <task-worktree>
git branch -d <merged-task-branch>
```

本次 `transfer_MAC` 的 commit-sync task worktree 在普通 remove 因子模組
拒絕後，已用上述方式移除；路徑不存在、本機與遠端任務分支均已清除。
`--force` 只處理已驗證的子模組限制，不代表可以丟棄其他人的未保存工作。

# 歷史上已驗證的非 force 替代流程

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
