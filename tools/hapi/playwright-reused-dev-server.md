---
title: "HAPI Playwright 會重用 5179 上別的 worktree 的 dev server"
scope: tools/hapi
status: active
updated: 2026-11-14
---

# HAPI Playwright 會重用 5179 上別的 worktree 的 dev server

`playwright.config.ts` 預設用 port 5179，並在非 CI 時設定 `reuseExistingServer`。如果另一個 worktree 已經在 5179 跑 vite dev server，`npx playwright test` 不會自己啟動 server，會直接連那一個。結果 `/e2e-fixtures/<新 fixture>.html` 拿到的是對方的 app（例如登入頁），測試以「找不到元素／等不到 download」這類看似無關的錯誤失敗。

解法：跑之前先 `ss -ltnp | grep 5179` 確認，或直接指定空閒 port：`PLAYWRIGHT_WEB_PORT=5287 npx playwright test e2e/<spec>`。2026-11-14 在 oracle 做 #2442 驗證時實際遇到並以此解決。
