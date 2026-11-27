---
title: Spec2RTL merged branch cleanup and develop retention
scope: projects/spec2rtl
status: verified snapshot; recheck live state before cleanup
updated: 2026-10-08
---

- Spec2RTL 由使用者選擇手動清理工作分支，保留 `main`、`develop`。查當前遠端 SHA、保護與所有 open PR 的 head/base，並用 `git merge-base --is-ancestor` 確認整支工作分支納入留存主線。PR `CLOSED` 或不同分支共用 SHA 都需同樣檢查。
- 刪除時用 `--force-with-lease=refs/heads/<branch>:<verified-sha>` 避免刪掉核對後新推的提交，刪後重查遠端。工作分支合併可用 `gh pr merge --match-head-commit <sha> --delete-branch`。常駐分支合併時保留，遠端清理不得連帶刪除未驗證的本機 worktree。
- 開啟自動刪 head 前須能保護 `develop`。2026-10-08 私有 repo 的 protection/rulesets API 回 HTTP 403，當時維持關閉。重查實際方案、授權和規則後再決定。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/spec2rtl/branch-cleanup.md)
