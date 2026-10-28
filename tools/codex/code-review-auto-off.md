---
title: Codex GitHub code review 自動審查已關閉
scope: tools/codex GitHub PR review
status: verified 2026-10-28
updated: 2026-10-28
---

- 設定頁：`https://chatgpt.com/settings/code-review`（舊連結 `/codex/cloud/settings/code-review` 會轉址）。「個人偏好設定 → 自動審查」已關閉，重新載入確認為 off。
- 所有 repositories 都是「自動程式碼審查：依個人偏好設定」，沒有 repo 層級覆寫，因此一個開關即全域生效。若之後新增 repo override，需逐一檢查。
- 原因：Codex review 額度常用盡，自動觸發只會在每個 PR 留下 `You have reached your Codex usage limits for code reviews`，誤導 agents。手動觸發：PR comment `@codex review`。
- 詳盡程式碼審查維持開啟、使用點數進行審查維持關閉（未改）。Cursor 對應設定見 `tools/cursor/bugbot-pr-review.md`。

## PR Review 停留 Running 的恢復（2026-10-28）

- Dashboard PR #2 的 summary 自 07:01 UTC 停在 Running；只看到 👀，不能當成審查通過，也不能據此判斷仍有後端工作執行。
- 確認 repository access 與 PR Review 設定後，15:43:51 UTC 在同一 head `8fe0a400a8ce34c229aab7bc0b03af7c8abee3e9` 再留一次 `@codex review`，15:44:56 UTC 得到 completed、無 findings，之後合併為 `2bc2698` 並部署。
- 這只證明同一 head 重新觸發成功；先前停留 Running 的後端根因未查明。不要宣稱關閉 Automatic review 修好了卡住問題：本次 Automatic review 全程維持 off。
- GitHub App repository access、PR Review 啟用與實際 review 完成是三個不同證據；Codex PR Review 與 Cursor Bugbot 是不同服務。避免把無回覆的手動指令重複洗版，先查 summary、設定及實際終態。
