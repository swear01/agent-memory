---
title: Brave CDP：target 讀回、attach 逾時與 Mac TCC
scope: tools/brave
tool: Brave Browser
machine: swairM5
status: active
created: 2026-08-29
updated: 2026-09-21
tags:
  - brave
  - cdp
  - chrome-devtools-protocol
  - cookie
  - automation
  - flatten-session
---

服務操作依使用者的瀏覽器選擇與授權。日常優先使用已授權的 connector 或現有內建瀏覽器；只有在已同意使用 CDP 的配置才開啟本機 debug endpoint。需要專用瀏覽器時保留使用者現有視窗、程序與 profile。僅停止已辨識歸屬的測試程序，不用廣泛 pattern 終止或刪原 profile lock。先以 loopback 的 `/json/version` 驗實際啟用的瀏覽器。

## 目標與 attach 失敗

- `/json` 第一個 page 不一定是目標。先按 URL、target ID 及實際頁面讀回核對。用 `/json/version` 的 browser-level WebSocket 呼叫 `Target.attachToTarget {targetId, flatten:true}`；後續 `Page`／`Runtime` 操作用返回的同一個 `sessionId`。重新 attach 後換回傳的新 ID；用錯會回 `Session with given id not found`。
- 重載用 attached session 的 `Page.reload`，不是 browser-level `Target.reload`。頁面 WS 連上但 `Runtime.evaluate` timeout 時，先分辨 target crashed、未完成頁面初始化或 JS dialog。用 `Page.javascriptDialogOpening` 確認對話框；只按實際任務和使用者授權處理，不自動接受所有 prompt。僅針對本次擁有的卡住 target 重建，並以實際頁面內容驗證。CDP access 不代表頁面或敏感權限已經取得。
- 不保存、匯出或在工具輸出顯示 Cookie；用實際頁面讀回驗證既有登入狀態。也不要用 browser profile 認證調查代替正式授權的登入入口。

## HAPI 語音連線與 TCC

從 HAPI runner 直接啟動的 Brave 曾被 TCC 歸責給 `xyz.hapi.cli`；該 build 缺 `com.apple.security.device.audio-input`。 `getUserMedia({audio:true})` 不返回，畫面卡 connecting、沒有 MediaRecorder 或 transcription POST，與同時切換 VPN 無因果證據。

停止該測試 CDP 程序，從 Dock／Finder／LaunchServices 正常啟動 Brave。已驗證的恢復條件是命令列不再帶 debug 旗標、原 9222 不監聽、系統記錄 `launchedByLS=1` 且 TCC responsible 為 `com.brave.Browser`；再以使用者授權的實際功能測試查復原。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/brave/cdp-remote-control.md)
