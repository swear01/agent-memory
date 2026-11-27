---
title: 家目錄、Documents 與雲端資料夾的工作檔邊界
scope: global/workflows
status: active
updated: 2026-11-28
tags: [home-directory, documents, cloud-drive, cwd, scratch, cleanup]
---

- Session 從家目錄啟動時，先確認 cwd，再把 review、渲染草稿、測試 log 放到任務的 scratch 或 `/var/tmp`。Task worktree 用既有 `<remote-home>/.agent-worktrees/`。Documents 頂層和課程雲端資料夾是使用者整理好的空間；交付物先找既有子資料夾，沒有適合位置才確認。
- 一次性 preview 用任務範圍內的程序，完成後查 PID 和 port 已停止。若接手已登錄的舊服務，先確認服務與本次任務相符，再從原 supervisor 停止。別留下 preview plist 或明文憑證備份。
- 清理前查 PATH、dotfiles、agent-memory 與 repo 歸屬，搬 repo 前看 dirty 和遠端分支。備份或同名不等於內容相同，用 `cmp` 驗證；獨有檔歸位，已確認可清理的檔案移到垃圾桶。永久清空由使用者決定。

[原始清理與課程檔案案例](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/global/workflows/keep-home-directory-clean.md)
