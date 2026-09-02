---
title: 2026-09-02 全域 linked worktree 清理
scope: projects/cpachecker
project: cpachecker
tags: [git, worktree, cleanup, stash, nfs]
status: verified
created: 2026-09-02
updated: 2026-09-02
---

# 結果

使用者明確要求清理所有額外 linked worktree，保留各 repository 的 primary checkout 與
`<remote-home>/cpachecker-experiments`。盤點共 34 個 secondary worktree：CPAchecker 29 個、
transfer_MAC 2 個、CPAchecker wiki 1 個、pono-llm 1 個、sv-benchmarks 1 個。

CPAchecker issue166 schema-11 worktree 的 claim owner session 仍顯示 active，但 cwd 已回到 main
repo 且任務完成；通知 owner 後，由該 session 自行移除 worktree、claim 與已由 `origin/main`
保存的 branch。其餘 33 個由 session-attached HAPI job `cleanup-all-worktrees` 逐一執行普通
`git worktree remove`，結果 `33/33 completed`。未使用 `--force`、`rm -rf` 或 reset。

# 保存的 dirty 狀態

7 個 dirty CPAchecker worktree 先以 `git stash push --include-untracked` 保存，stable stash commit
如下；恢復前可用 `git -C <remote-home>/cpachecker stash show -p <SHA>` 檢查，再在合適的 clean
checkout 執行 `git stash apply <SHA>`：

- `c1d9b06ccc4bd6627c2efd2197b5c15e3c26001c` — issue166 generalization confirmatory
- `aa34bd2104ca7c21cf1c702f5919a7f39d882247` — inherited guard adaptive
- `48b56a8e06c0af873e44f9eb7ac0525487a169fe` — issue136 prompt hypothesis
- `1b33d3651d551981a41d1bb9545c3daf22947210` — flash-low-300s-v2
- `1c17b610ce08277bfb1152c1ad4bbd4a66a64c59` — issue102 replay smoke flash
- `bd504df2c81da897d635c9e107c1c7b43aa574c1` — issue102 replay smoke stock
- `7dcc2b26a12a6351a67da6143025196c32b31da3` — Claude agent worktree

Clean worktree 的未合併 local branches 均保留；worktree 移除不刪 branch。兩個 transfer_MAC
worktree 的 initialized-submodule metadata cache 依已驗證流程搬到可恢復的 Trash：

`<remote-home>/.local/share/Trash/files/worktree-cleanup-20260902/submodule-caches/`

13 個已失效 claim 也移到同一 Trash 的 `claims/`；schema-11 claim 由原 owner 移除。

# 最終驗證

- 全域掃描已知 home、中央目錄、Skillshare、plugin cache、`/tmp`、`/var/tmp` 的 Git common
  directories，沒有任何 registry 含第二個 worktree。
- `~/.agent-worktrees` 內 `.git` worktree pointer 為 0；舊的 `~/cpachecker-runs`、
  `~/cpachecker-worktrees`、home sibling、repo 內 `.claude/worktrees` 與 pono `.worktrees` 路徑為 0。
- 中央 claim 為 0，活動 process cwd 指向已移除 worktree 的數量為 0。
- `cpachecker-experiments/runs` 清理前後均為 69 個目錄，未被清理。
- primary repositories 保留。CPAchecker main 最終仍有原本的 1 筆 working-tree change；
  transfer_MAC source 仍是既有 dirty checkout，沒有把它當 secondary worktree 移除。
- `~/.agent-worktrees` 仍有 transfer_MAC source 與 5 個乾淨的 CPAchecker wiki primary clones；
  它們各自只有 primary checkout，不是額外 linked worktree。
