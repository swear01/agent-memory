---
title: Brave 有頭 CDP 遠端控制（cookie 讀寫、頁面操作）
scope: tools/brave
tool: Brave Browser
machine: swairM5
status: active
created: 2026-08-29
updated: 2026-08-30
tags:
  - brave
  - cdp
  - chrome-devtools-protocol
  - cookie
  - automation
---

# 啟動（使用者 Mac，Apple Silicon）

> **TCC 警告：** 從 HAPI runner／agent 工作階段直接執行 Brave，macOS 可能把 Brave 的 privacy responsibility 歸到 `xyz.hapi.cli`。目前 HAPI macOS release binary 使用 hardened runtime，但沒有 `com.apple.security.device.audio-input` entitlement；此 CDP Brave 不適合測試麥克風或其他 TCC-sensitive 功能。

1. 請使用者 ⌘Q 完全退出 Brave
2. `pkill -f "remote-debugging-port=9222"; rm -f ~/Library/Application\ Support/BraveSoftware/Brave-Browser/Singleton{Lock,Socket,Cookie}`
3. `"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --remote-debugging-port=9222 --no-first-run &`
4. `curl http://127.0.0.1:9222/json/version` 驗證；`/json` 拿目標 tab 的 `webSocketDebuggerUrl`

# 要點（2026-08 驗證，Brave/Chromium 152，Node 26 全域 WebSocket）

- E 站（e-hentai/exhentai）有 Cloudflare Turnstile：**headless 被擋、有頭模式約 2 秒過**
- 連 `ws://127.0.0.1:9222/devtools/page/<id>`；模式＝`pending Map + cmd(method, params) Promise 封裝 + onopen 內 async/await 順序執行`
- 讀 cookie：`Network.enable` → `Network.getAllCookies`
- 寫 cookie：`Network.setCookie {name, value, domain, path, expires, httpOnly}`；domain 要寫 `.exhentai.org` 這種含前綴點才會被子網域共用
- 刪 cookie：`Network.deleteCookies {name, domain, path}`（domain/path 照 getAllCookies 回傳原樣傳）
- 跳轉：`Page.navigate {url}` + sleep 等載入；讀內容：`Runtime.evaluate {expression, returnByValue: true}`
- **挑 tab 要依 URL 過濾**：`/json` 的第一個 page 不一定是要操作用的 tab
- E 站自動登入：刪掉 `ipb_member_id`/`ipb_pass_hash` 後下次載入會自動重新登入並重寫 cookie（含 `star`/`hath_perks`），所以「登出」狀態留不住；重新登入會拿到最新捐款旗標
- 搜尋結果表 selector 不穩（新版 UI 不一定有 `#gt`）→ 改用 `a[href*="/g/"]` 抓標題樣本驗證
- 用畢 ⌘Q 再正常重開即可，cookie/登入狀態全保留

# HAPI Standard 語音卡在 connecting（2026-08-30，swairM5）

已驗證的因果鏈：正常啟動的 Brave 由 TCC 歸責給 `com.brave.Browser`；HAPI runner 管理的 Pi 工作階段從 `playground` 直接執行上述 CDP Brave 後，TCC 改歸責給 `xyz.hapi.cli`，並記錄 responsible process 缺少 `com.apple.security.device.audio-input`。此時 `navigator.mediaDevices.getUserMedia({audio:true})` 不 resolve/reject，HAPI Standard 前端又沒有 timeout，因此 UI 永久顯示 `connecting`；流程尚未建立 MediaRecorder，也不會送出 transcription POST。

不要僅憑「連 VPN 後失敗」判斷網路問題：本次 DVLab IKEv2 連線與正常 Brave／CDP Brave 的程序替換同時發生。DNS、TLS、proxy/PAC、route、MTU、SSE、Service Worker 與 HAPI API 均正常；真正變項是 browser process 的 TCC responsibility。

CDP 使用完畢後，必須完全退出並由 Dock／Finder／LaunchServices 正常重開，再測試麥克風。恢復判據：Brave command line 沒有 `--remote-debugging-port`、本機 9222 未監聽、系統紀錄為 `launchedByLS=1`，且 TCC `responsible` 為 `com.brave.Browser`。2026-08-30 手動重開後已符合這四項；未保存或上傳任何測試錄音。
