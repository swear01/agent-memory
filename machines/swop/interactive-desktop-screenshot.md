---
title: swop 透過 SSH 擷取真實桌面截圖（互動式排程工作）
scope: machines/swop
machine: swop
status: active
updated: 2026-11-04
---


`ssh swop` 落在非互動 session，`MainWindowTitle` 可能空白，`CopyFromScreen` 可能產生黑圖。

擷取前確認使用者授權，再用 `Get-Process explorer -IncludeUserName` 確認目前的互動使用者和 SessionId。在同一使用者的桌面執行核准的腳本，使用一次性 `schtasks /create … /it /ru <interactive-user>` 排程，完成後刪除工作。核對工作執行結果與實際截圖。

複雜 PowerShell 查詢透過 SCP 上傳 .ps1 再用 `powershell -File` 執行，避免 SSH 的 Windows command shell 改寫 `$_` 或引號。

[原始驗證紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/swop/interactive-desktop-screenshot.md)
