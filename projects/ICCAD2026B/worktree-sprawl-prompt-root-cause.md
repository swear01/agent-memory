---
title: ICCAD2026B worktree 散落的 prompt 根因
scope: projects/ICCAD2026B
project: ICCAD2026B
tags: [git, worktree, agent-rules, cursor, skillshare]
status: verified
created: 2026-09-02
updated: 2026-09-02
---

- **多 agent worktree 散在主目錄，或清掉後又生成。** 先用 `git worktree list --porcelain`、第一筆 reflog 與 child prompt 查路徑來源。歷史 prompt 只給 branch／worktree 名稱，child 依舊 sibling 範例自行選路徑；早期留存不等於目前仍在建立。
- **全域要求中央 worktree，但某個 skill 還指向 /tmp。** 查實際安裝的 skill、repo tests 及範例；歷史 `fix-issue` 和測試同時要求舊 /tmp，改成 `~/.agent-worktrees/<repo>-<path-hash>/issue-<number>` 才能修真正入口。父任務交辦時寫明完整路徑；同 task worker 並行與多 issue fan-out 是不同授權。
- **普通 worktree remove 因 initialized submodule 失敗。** 先驗 ownership、dirty 和活躍程序；依 [submodule-safe cleanup](../../tools/git/worktree-submodule-cleanup.md) 走可驗證的非強制處理，失敗保留。舊專案一次性刪除規模見歷史，不從那次授權推定可清現場其他 repo 或快取。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ICCAD2026B/worktree-sprawl-prompt-root-cause.md)
