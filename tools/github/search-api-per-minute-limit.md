---
title: gh search 連打會撞 Search API 每分鐘 30 次上限，不是 core 額度用完
scope: tools/github
status: verified
updated: 2026-11-27
tags: [gh, github-api, rate-limit, search]
---

2026-11-27 在 swever 用迴圈連續跑約 25 次 `gh search issues --repo tiann/hapi ...`，後段回 `HTTP 403: API rate limit exceeded for user ID ...`。已登入（OAuth `gho_` token），錯誤訊息看起來像整個帳號額度爆掉，但隨即查 `gh api rate_limit`：core 4845/5000、graphql 4975/5000，search 已重置為 30/30。實際撞到的是 Search API 的每分鐘上限（已驗證身分 30 次/分鐘；code search 10 次/分鐘）。

- 先跑 `gh api rate_limit --jq '.resources|{core,search,graphql}'` 判斷是哪個 bucket，這個查詢本身不計額度。
- 連續搜尋每次間隔 ≥2.5 秒，或把多個關鍵字合併成較少的查詢；撞到時照 `retry-after`，沒有就等 60 秒，不要立刻重試。
- 額度以使用者計算：同帳號在整個 fleet 上的 agent 共用同一份（文件寫 personal rate limit，跨 token 共用屬推論）。GitHub App 只對安裝的 repo 有獨立額度，對上游別人的 repo 和 Search 上限都沒幫助。
- 等 CI／bot review 時用 `gh pr checks --watch -i 30` 之類的固定間隔，不要密集輪詢。

依據：docs.github.com REST rate limits、REST search `#rate-limit` 與 best practices 頁面。
