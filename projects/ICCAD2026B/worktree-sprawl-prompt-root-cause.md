---
title: ICCAD2026B worktree 散落的 prompt 根因
scope: projects/ICCAD2026B
project: ICCAD2026B
tags: [git, worktree, agent-rules, cursor, skillshare]
status: verified
created: 2026-09-02
updated: 2026-09-02
---

# 結論

ICCAD2026B 的大量 worktree 不是 Git 自動產生，而是 2026-07 至 2026-08 的多 agent
工作流留下。2026-09-02 盤點 `git worktree list --porcelain` 共 91 筆：main 1、
`~/.agent-worktrees/` 4、home 直下 repo sibling 58、repo 內 `build/worktrees/` 3、
`/var/tmp/` 25。

58 個 home sibling 的 first reflog 建立時間全部落在 2026-07-17 至 2026-08-07；
沒有任何一個是在中央路徑政策生效後建立。ICCAD2026B 的 `AGENTS.md` 在 commit
`d29d02a7`（2026-07-22）加入「Maximum useful concurrency」，58 個中有 56 個在此後
建立。Cursor child transcript 證實 2026-08-07 同一時間派出多個 issue agent；child prompt
只指定 branch/worktree 名稱，未指定絕對路徑，agent 遂自行使用
`<remote-home>/ICCAD2026B-issueNNN`。

# 規則時間線與目前缺口

- `personal-pr-workflow` 初版 commit `e6ffecc`（2026-08-02）範例明寫
  `git worktree add <sibling-path> ...`，直到 `3420d0f`（2026-08-20）才改成
  `~/.agent-worktrees/<repo-key>/<task>`，並禁止 sibling、`/tmp` 與 repo 內路徑。
- global agent rule 在 transfer_MAC commit `5c389d6`（2026-08-23）才加入中央路徑要求；
  所以歷史 home sibling 不代表目前規則仍在建立新 sibling。
- 目前仍有真實衝突：installed `fix-issue/references/worktree-and-pr.md` 明寫
  `WORKTREE="/tmp/${REPO_SLUG}-issue-${ISSUE_NUMBER}"`，而 transfer_MAC
  `tests/test_fix_issue_skill.py` 還反向測試必須使用該 `/tmp` 路徑。這會覆蓋較泛用的 global
  規則，必須與 `personal-pr-workflow` 對齊。
- `personal-pr-workflow` 雖要求 merge 後普通 `git worktree remove`，卻未處理 initialized
  submodule。ICCAD agent transcript 已記錄 merge 後 cleanup 因 submodule 被 Git 拒絕，且依
  no-force 規則正確保留 worktree。安全非 force 流程已記在
  `tools/git/worktree-submodule-cleanup.md`，未來 prompt 應引用該流程。

# 最小修正方向

1. 把 `fix-issue` 的 `/tmp` 路徑與對應 test 改成中央 root，並加入一個全域 contract test：
   installed skills 不得出現 task worktree 的 sibling、repo 內或 `/tmp` fallback。
2. 收斂 ICCAD 的 concurrency 規則：允許同一 task 內的 worker/process 並行，不得因此自動
   展開多個 issue/task worktree；多 task fan-out 必須由使用者明確要求。
3. 在共用 cleanup lifecycle 加入 clean/ownership 驗證與 submodule-aware 非 force 清理；
   任一步無法驗證才保留並回報。

