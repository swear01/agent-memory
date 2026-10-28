---
title: Zeus 無系統 Chromium，截圖驗證改用 Playwright 內建 chromium
scope: machines/zeus
machine: zeus
status: active
created: 2026-10-29
updated: 2026-10-29
tags:
  - browser
  - screenshot
  - verification
---

- zeus（swear02）沒有 `chromium`／`chromium-browser`／`google-chrome`；但 `~/.cache/ms-playwright/` 已有 `chromium-1228`（另有 `chromium_headless_shell-1228`）。
- 前端改動的截圖驗證可直接用：`~/.cache/ms-playwright/chromium-1228/chrome-linux64/chrome --headless --no-sandbox --window-size=1400,900 --screenshot=<out.png> <url>`。沒有 `--no-sandbox` 會失敗。
- 不想動正式站時，把頁面與 runtime JSON 複製到暫存目錄，用 `python3 -m http.server <port>` 起本地伺服器再截圖；2026-10-28／29 驗證 DVLab dashboard treemap 與硬碟橫條時這樣做有效。
- 版本號 `1228` 會隨 Playwright 更新而變；先 `ls ~/.cache/ms-playwright/` 再組路徑。
