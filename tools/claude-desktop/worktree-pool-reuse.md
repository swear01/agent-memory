---
title: Claude 桌面版會把已歸檔 session 的 worktree 重新分給新 session
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
evidence: 2026-10-31 app log（`[WorktreePool] Reused worktree ...`、`[rebindWorktree] Rebound ...`）與 `ccd_session_mgmt list_sessions` 的 `cwd` 對照查證；Verto 專案實際發生。
created: 2026-10-31
updated: 2026-10-31
tags:
  - claude-desktop
  - worktree
  - sessions
---

# worktree 目錄名稱不代表現在是誰在用

## 現象

- 桌面 app 把 session worktree 放在 `<project-root>/.claude/worktrees/<name>-<hash>`。session 歸檔後，worktree 會「停放」在池子裡；之後開新的 session（例如 `start_session` 的 side session）時，app 可能直接拿這個舊目錄來用，並切到新 branch，目錄名稱不會改。
- log 長這樣：
  - `[WorktreePool] Reused worktree level16-chaos-4dd1b5 for session local_<id> (was leased by none, untouched since parked)`
  - `[rebindWorktree] Rebound <project-root>/.claude/worktrees/level16-chaos-4dd1b5 ... on branch cld/level-b-redesign-218cee`
- 所以 `level16-chaos-*` 這個 worktree 裡的未提交改動，其實是另一個 session「Redesign level-b」正在做的工作，跟 level16 無關。

## 動 worktree 之前

- 先用 `mcp__ccd_session_mgmt__list_sessions` 找出 `cwd` 等於該 worktree 的 session（`get_session` 的 `worktreePath` 也可以），再看 `git -C <worktree> branch --show-current`。不要從目錄名稱推斷擁有者。
- 有現任 session 在用的 worktree，不要 stash、checkout、clean 或 remove，即使那個 session 沒在跑。
- 2026-10-31 的錯誤：想在歸檔前保住改動，就把 `level16-chaos-*` 的未提交工作 `git stash -u` 了，結果動到另一個 session 的進行中工作；一分鐘內 `git stash pop` 原樣還原。教訓：先查擁有者，再決定要不要碰。
- 補充：`git stash` 存在共用的 `.git`，worktree 被刪掉 stash 還在；但 pop 時要回到原本的 worktree。

## 清理已歸檔 session 留下的 dirty worktree

- 沒有現任 session 的 dirty worktree，先判斷改動是否還需要：比對 `HEAD` 與 `origin/main`。Verto 的例子是 `Assets/Phase/Resources/Levels/*.png` 重新 bake 的關卡縮圖（產生物），main 之後已重新 bake，且部分關卡已被刪除 → 不需要，`git worktree remove --force` 後 `git worktree prune`。
- PNG 是否「只是重新編碼」不能用 `sips` 轉 BMP 後 `cmp` 判斷（色彩格式不同也會不一樣）；用 `uv run --no-project --with pillow==11.0.0` 轉 RGBA 逐像素比。
- 本機 branch 若等於該 PR 的 `headRefOid` 且 PR 已 MERGED（squash 合併時 `origin/main..branch` 仍會列出 commit），可以 `git branch -D`。
