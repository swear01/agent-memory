---
title: Cursor Bugbot 的 usage limit 留言與手動審查
scope: tools/cursor GitHub PR review
status: verified
updated: 2026-11-28
---

看到 `Bugbot couldn't run - usage limit reached` 時，先查留言對應哪個觸發與 head SHA。帳號若開著 `Every Push`，額度不足時每次 push 都可能留下這句。該留言只證明有一次嘗試，合併前仍要驗證最新 head 有完成且通過的 review。

- 2026-10-22 重新載入設定頁驗證為 `Manual Only`，PR Summaries 為 `Off`。審查依 `github-pr-review-loop` 選擇 provider；選 Bugbot 時用 `bugbot run` 或 `@cursor review` 手動觸發。
- 診斷觸發問題時，分別核對 GitHub App 的 repository access、Cursor 的 repository 開關和帳號層級 Trigger Mode。設定保存和最新 head 的審查結果應分開查證。

[原始設定與驗證紀錄](https://github.com/swear01/agent-memory/blob/d1e08b2dad393fd2ecf9dbbee4f69d53bd6d8755/tools/cursor/bugbot-pr-review.md)
