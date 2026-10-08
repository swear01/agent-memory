---
title: Spec2RTL merged branch cleanup and develop retention
scope: projects/spec2rtl
status: verified snapshot; recheck live state before cleanup
updated: 2026-10-08
---

## 清理原則與方案限制

- 保留常駐分支 `main`、`develop`。工作分支清理前重新查詢遠端 SHA、分支保護和開啟中的 PR；不得刪除其他 PR 的 head 或 base。用 `git merge-base --is-ancestor` 確認整支分支的提交已包含在保留的主線，再用 `--force-with-lease=refs/heads/<branch>:<verified-sha>` 防止刪掉核對後新增的提交，刪除後重新查詢 GitHub。
- PR `CLOSED` 不代表 `MERGED`；只憑 PR 關閉、分支名稱或工作已完成不能判定可刪。不同分支若指向同一 SHA，仍須確認該 SHA 已進主線且沒有開啟中的 PR 使用它。
- 2026-10-08 實查 `DVLab-NTU` 為 Free 組織，`spec2rtl` 為 private repo；rulesets 和 `develop` branch protection API 均回傳 HTTP 403，要求升級或公開 repo。保留組織 private repo 時，需 GitHub Team 或以上才能使用分支保護／rulesets。
- GitHub 的 `Automatically delete head branches` 可搭配禁止刪除的保護規則保留 `develop`；目前無法設定該保護，因此 `delete_branch_on_merge=false` 維持關閉。此次使用者選擇手動清理，沒有升級方案、公開 repo、啟用自動刪除或新增清理 Action。
- `personal-pr-workflow` 已要求工作分支合併時使用 `gh pr merge --match-head-commit <sha> --delete-branch`；常駐分支合併須保留來源。遠端清理不等於本機 worktree 清理；歸屬、是否活躍或乾淨狀態未確認時保留 worktree。

## 2026-10-08 清理快照

- 本次手動刪除並讀回確認七支遠端分支：`ci/content-safety-20261008`、`ci/regression-flow`、`integrate`、`skills/parallel-verification`、`sync/skill-rtl-main`、`sync/skill-rtl-to-develop`、`architecture`。
- `sync/skill-rtl-main` 的 PR #7 雖關閉未合併，但與已合併 PR #8 的來源指向相同 SHA，且已包含在 `develop`。`architecture` 的全部提交已包含在 `main` 與 `develop`，沒有獨有提交。
- PR #13 在本次對話期間由其他工作合併，`ci/parallel-workers` 隨後已不存在；沒有把這支計入本次手動刪除七支。
- 最終保留 `main`、`develop`、`ci/cursor-sdk-smoke`、`feat/swear-review`、`ppa`，當時沒有開啟中的 PR。後三支相對 `main`、`develop` 分別仍有 3、1、1 個未合併提交，故保留。本機 worktrees 和工作檔案未刪除，repo 設定未變更。
- 以上是當日快照，後續必須重查 GitHub 分支、PR、SHA 和方案，不可沿用快照直接刪除。
