---
title: swop 透過 SSH 擷取真實桌面截圖（互動式排程工作）
scope: machines/swop
machine: swop
status: active
updated: 2026-11-04
---

# swop：SSH 下擷取桌面截圖

- `ssh swop` 跑在非互動 session，`Get-Process` 看不到 `MainWindowTitle`，`CopyFromScreen` 會得到黑圖。
- 使用者 `stanl` 通常已登入互動桌面（`Get-Process explorer -IncludeUserName` 可看到 SessionId）。把下載／啟動／UI Automation／`CopyFromScreen` 寫成 ps1，用 `schtasks /create ... /it /ru stanl` 建立互動式排程工作跑一次，完成後刪除，就能拿到真實畫面。
- 2026-11-04 用此法下載 MobaXterm Home Edition portable（v26.6）到 `<swop-home>/Documents/Codex/2026-11-04/mobaxterm/`，以 gupstest 金鑰實連工作站並截圖 Session settings、X11-Forwarding、`DISPLAY=localhost:10.0`、xterm。
- v26.6 的 SSH 表單是 `Remote host` / `Username` / `Port`，沒有舊教學說的「Specify username」勾選框；X11-Forwarding 與 Compression 在 `Advanced SSH settings` 分頁。
- 在 ssh 裡用 `powershell -Command` 傳 `$_`、引號很容易被 cmd 吃掉，複雜查詢改成 scp 一個 ps1 再 `-File` 執行。
