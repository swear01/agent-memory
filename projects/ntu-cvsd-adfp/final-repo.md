---
title: CVSD 期末專題（SCE）獨立 repo
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-11-23
---

# CVSD 期末專題（SCE）獨立 repo

- 2026-11-23 起，期末專題放在獨立的 private repo `swear01/cvsd-final-sce`，分支 `main`。組員 No-one-Here1111 有 write 權限。
- 歷史是用 `git filter-repo --path final/ --path-rename final/:` 從 CVSD2026 的 `final-sce` 分支切出來的：保留 143 個 commit，`final/` 變成 repo 根目錄，所以 commit hash 全部換了。CVSD2026 遠端上的 `final-sce` 分支已刪除。
- 本機 clone：`~/.agent-worktrees/cvsd-final-sce`。舊的 worktree `cvsd-final.migrated-backup` 只是備份，不要在那裡工作。
- 舊的腳本、consult 紀錄和遠端主機（gups/cthulhu）上的同步路徑如果寫死了 `.../cvsd-final/final/...`，要改成新路徑，並去掉 `final/` 這層。
- 和組員分享時只給期末 repo，不要給 CVSD2026，因為裡面有個人的 HW/lab 作業。
