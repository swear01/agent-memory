---
title: "git stash 跨 worktree 共用，平行 session 不可用 stash 做 baseline"
scope: "tools/git"
status: active
updated: 2026-10-22
---

# git stash 跨 worktree 共用

## Problem

2026-10-22 在 hapi 平行 session 中，本 session 於自己的 worktree 執行 `git stash push <file>` → 跑測試 → `git stash pop`。同時另一個 session 在別的 worktree 做了同樣動作。stash 是 repository 共用（refs/stash 在 common git dir），結果兩邊 pop 到對方的 stash：本 session 的 `codexSessions.ts` 修改跑到 `issue-2045-tool-only-tail`，對方的 `message-window-store.ts` 修改跑到本 worktree。對方 session 自行修復；本 session 以 diff 比對確認其內容完全一致後才丟棄本地副本並重新套用自己的修改。

## Durable rule

多個 session 共用同一 repository 的 worktree 時，不要用 `git stash` 暫存修改（包括驗證「測試在未修正時失敗」）。改用 `git diff > /tmp/<task>.patch && git checkout -- <paths>`，跑完再 `git apply /tmp/<task>.patch`，或在 upstream base 另開 detached worktree 跑 baseline。

## Recovery

若 pop 後 `git status` 出現不屬於本任務的檔案：先停止，`git worktree list` 掃描所有 worktree 的 status 找出交換對象，各自 `git diff` 存成 patch，比對確認後再還原；不要直接 checkout 掉對方的修改。
