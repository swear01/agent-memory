---
title: Codex GitHub code review 自動審查已關閉
scope: tools/codex GitHub PR review
status: verified 2026-10-22
updated: 2026-10-22
---

- 設定頁：`https://chatgpt.com/settings/code-review`（舊連結 `/codex/cloud/settings/code-review` 會轉址）。「個人偏好設定 → 自動審查」已關閉，重新載入確認為 off。
- 所有 repositories 都是「自動程式碼審查：依個人偏好設定」，沒有 repo 層級覆寫，因此一個開關即全域生效。若之後新增 repo override，需逐一檢查。
- 原因：Codex review 額度常用盡，自動觸發只會在每個 PR 留下 `You have reached your Codex usage limits for code reviews`，誤導 agents。手動觸發：PR comment `@codex review`。
- 詳盡程式碼審查維持開啟、使用點數進行審查維持關閉（未改）。Cursor 對應設定見 `tools/cursor/bugbot-pr-review.md`。
