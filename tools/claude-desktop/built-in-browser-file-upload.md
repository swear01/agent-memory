---
title: Claude 桌面版內建瀏覽器無法上傳本機檔案；改用剪貼簿放檔案讓使用者貼上
scope: tools/claude-desktop
tool: Claude desktop app built-in browser
status: active
confidence: high
evidence: 2026-11-21 在 messenger.com 實測：原生選檔視窗碰不到、window.open 取代原分頁、即使 CSP 放行 localhost 仍 ERR_BLOCKED_BY_CLIENT；剪貼簿檔案由使用者在 Brave 貼上成功。
created: 2026-11-21
updated: 2026-11-21
tags:
  - claude-desktop
  - built-in-browser
  - file-upload
  - messenger
---

# 內建瀏覽器無法上傳本機檔案

搜尋關鍵字：內建瀏覽器上傳檔案、附件、Messenger 傳 PDF、file input、file chooser、ERR_BLOCKED_BY_CLIENT、localhost。

在 Claude 桌面版的內建瀏覽器（`mcp__Claude_Browser__*`）裡，網頁要附加本機檔案時，三條路都不通：

1. 點附件按鈕會開 Claude.app 自己的原生選檔視窗；computer-use 的可授權 app 清單沒有 Claude 本身，所以操作不到。
2. `window.open(...)` 不會開新視窗，而是把目前分頁導走，無法用 opener + `postMessage` 從另一頁轉送檔案。
3. 從網頁 `fetch` 本機伺服器（`http://localhost:*`、`127.0.0.1`）一律 `net::ERR_BLOCKED_BY_CLIENT`，是瀏覽器刻意阻擋；就算網站 CSP 放行（Messenger 的 `connect-src` 有 `http://localhost:3103`）也一樣。不要再繞。

剩下把檔案轉 base64 分批灌進頁面的做法，對幾百 KB 的檔案太慢太耗 token，不建議。

## 可行做法

把檔案以「檔案」形式放到 macOS 剪貼簿（等同 Finder 拷貝），請使用者在自己的瀏覽器聊天框按 Cmd+V 再送出：

```bash
F="<absolute path to file>"
osascript -l JavaScript -e "ObjC.import('AppKit'); var pb=\$.NSPasteboard.generalPasteboard; pb.clearContents; pb.writeObjects(\$([\$.NSURL.fileURLWithPath('$F')]))"
```

只用 `set the clipboard to (POSIX file ...)` 只會放 `furl`，不一定能被瀏覽器當附件貼上；`writeObjects` 會同時放 `public.file-url` 與 `NSFilenamesPboardType`。需要好看的檔名就先 `cp` 成新名字再放。

另外：內建瀏覽器的 Messenger 需要使用者自己登入，Facebook 可能跳「檢查最近的登入紀錄」，那是帳號安全確認，交給使用者按。
