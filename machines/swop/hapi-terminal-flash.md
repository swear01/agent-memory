---
title: HAPI 排程啟動造成 Windows Terminal 閃窗
scope: machine
status: verified
updated: 2026-09-06
---

swop 的 HAPI Runner 排程使用 Interactive 登入模式，每分鐘執行 PowerShell，帶有 -WindowStyle Hidden。2026-09-06 兩次程序監看確認：排程先建立 powershell.exe、conhost.exe、OpenConsole.exe、WindowsTerminal.exe，約一秒後才啟動 hapi.exe。因此初始終端機閃窗發生在 HAPI 執行之前。

升級後存活的 Runner 已不由原排程持有；排程為 Ready，IgnoreNew 無法阻止每分鐘重跑。重跑日誌為 Runner already running with matching version，成功退出，並非 Runner 崩潰。

v0.29.0.6 的 cli/src/utils/process.ts 程序查詢已使用 windowsHide: true；不能因看見查詢 PowerShell 就斷言漏加此旗標。修正應優先驗證排程入口在建立程序時隱藏視窗，保留每分鐘復原與現有 session；本次僅診斷，未實作或驗證修正。
