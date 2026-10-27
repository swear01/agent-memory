---
title: "spawn-peer 要用 --session-type worktree，session 才會歸在 repo 底下"
scope: "tools/hapi"
status: active
updated: 2026-10-27
---

# spawn-peer 要用 --session-type worktree，session 才會歸在 repo 底下

自己 `git worktree add` 到 `~/.agent-worktrees/<repo>/<ws>`，再用 `--dir` 指向它開 simple session，HAPI UI 會把每個 worktree 當成獨立專案，不會歸在 repo 底下（使用者回報 session「沒在正確位置」）。

做法：`hapi spawn-peer --dir <repo 主目錄> --session-type worktree ...`。HAPI 會從 repo 目前的 HEAD 建 `<repo>-worktrees/<MMDD-xxxx>`、branch `hapi-<MMDD-xxxx>`，並在 metadata 記 `worktree.basePath`，UI 依此分組（2026-10-27 在 mazu 驗證路徑和 branch；`inspect-peer` 不顯示 worktree metadata）。使用者要求分組時，這個規則優先於「worktree 放在 ~/.agent-worktrees」。

相關行為（同日驗證）：
- `spawn-peer --message-file -` 前面不要加 `unbuffer`：stdin 被換成 tty，指令會一直卡住。改用檔案並加上 `</dev/null`。
- `ping-peer` 送給正在跑的 session 時，訊息會排隊等目前這輪結束；要立刻處理就先 `abort-peer`。
- `archive-peer` 不會殺掉 session 用 setsid 起的背景工作。
- `wait-peer` 回 `status: completed` 只代表這輪結束；session 若在等背景工作，同時會是 `active: true`，之後背景工作完成會自己再開一輪。要拿最終結果就再發 remit（ping-peer）並 wait 新的 remit ID。
- 跑中的 session 不能改目錄；要搬就請它 commit、寫交接筆記，封存後再開新的。
