---
title: swairM5 沒有 Chrome，但有 Chromium 與 Brave 可做無頭渲染
scope: machines/swairM5
machine: swairM5
tool: chromium
status: active
confidence: high
created: 2026-11-06
updated: 2026-11-06
tags:
  - browser
  - headless
  - screenshot
  - verification
---

- swairM5 沒有 Google Chrome，但有 `/Applications/Chromium.app`（156.0.8078.47）和 `/Applications/Brave Browser.app`（156.1.98.63），兩者都是 Chromium 核心，旗標相同。**不要因為找不到 Chrome 就說無法渲染驗證**；使用者把這種回報視為重大錯誤。
- 找瀏覽器先用 `mdfind "kMDItemCFBundleIdentifier == 'org.chromium.Chromium' || kMDItemCFBundleIdentifier == 'com.brave.Browser'"`，或看 `/Applications`；zsh 的 glob 沒匹配會讓整行 `ls` 失敗，改用 `find`。
- 已驗證可用（2026-11-06，本機 HTML 行程頁）：
  `"/Applications/Chromium.app/Contents/MacOS/Chromium" --headless=new --disable-gpu --user-data-dir="$(mktemp -d)" --virtual-time-budget=8000 --window-size=1300,2400 --screenshot=out.png "file:///path/page.html#anchor"`
  `--dump-dom` 可取 JS 執行後的 DOM；加 `--enable-logging=stderr --v=0` 可在 stderr 看到 `INFO:CONSOLE` 的 JS 錯誤。`CVDisplayLinkCreateWithCGDisplay failed` 是無頭模式雜訊，可忽略。
- 頁面在 Google Drive 路徑時，先複製到 scratchpad 再用 `file://` 開。
- Brave：同樣旗標在 2026-11-06 測試時（使用者的 Brave 正在執行，另給獨立 `--user-data-dir`）`--headless` 與 `--headless=new` 都卡到 timeout、沒產生截圖；需要截圖時先用 Chromium。Brave 有頭 CDP 控制見 [[cdp-remote-control]]。

來源：2026-11-06 沖繩行程 HTML 渲染驗證。
