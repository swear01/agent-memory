---
title: gh api POST 遇到 connection reset 時可能其實已成功
scope: tools/github
status: verified
updated: 2026-10-22
tags: [gh, github-api, review-comments, retry, idempotency]
---

用 `gh api repos/<owner>/<repo>/pulls/<n>/comments/<id>/replies -f body=...` 回覆 review 討論串時，指令曾回報 `read tcp ... connection reset by peer` 並以 exit code 1 結束（2026-10-22，swear01/Verto PR #6）；但查詢 `pulls/<n>/comments` 發現那則回覆其實已經建立。

- 非冪等的 POST（留言、回覆、建立 PR 等）失敗時，不要直接重試；先查證是否已存在，例如 `gh api "repos/<owner>/<repo>/pulls/<n>/comments?per_page=100" --jq '[.[]|select(.in_reply_to_id==<id>)|.id]'`，否則會留下重複留言。
- 批次回覆多個討論串時，先抓出已有自動回覆的 `in_reply_to_id` 清單並跳過，整批就可以安全重跑。
- 解決討論串用 GraphQL `resolveReviewThread`（threadId 由 `reviewThreads` 查詢裡第一則留言的 `databaseId` 對應出來）；這個 mutation 重複執行沒有副作用。
