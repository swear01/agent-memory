---
title: HAPI 登入後排程無視窗啟動與復原
scope: machine
status: verified
updated: 2026-09-07
---

swop 的 HAPI Runner 排程使用 Interactive 登入模式，每分鐘執行 PowerShell，帶有 -WindowStyle Hidden。2026-09-06 兩次程序監看確認：排程先建立 powershell.exe、conhost.exe、OpenConsole.exe、WindowsTerminal.exe，約一秒後才啟動 hapi.exe。因此初始終端機閃窗發生在 HAPI 執行之前。

升級後存活的 Runner 已不由原排程持有；排程為 Ready，IgnoreNew 無法阻止每分鐘重跑。重跑日誌為 Runner already running with matching version，成功退出，並非 Runner 崩潰。

v0.29.0.6 的 cli/src/utils/process.ts 程序查詢已使用 windowsHide: true；不能因看見查詢 PowerShell 就斷言漏加此旗標。修正應在排程入口建立程序時隱藏視窗，保留每分鐘復原與現有 session。以下修正已於 2026-09-07 部署並驗證。

## 已部署方案

使用者選擇保留無密碼帳戶的 InteractiveToken 登入後排程，沿用原本工具、憑證與桌面環境；必須先登入 Windows。不要把此決定改成 WinSW、SYSTEM、S4U 或另建服務帳戶。服務登入驗證曾遇到 1069/1326；驗證服務已移除，WinSW 未接管正式 Runner。

只替換排程 Action：以 `%SystemRoot%\System32\wscript.exe` 加上 `//B //Nologo` 執行 `%LOCALAPPDATA%\Programs\Hapi\start-hapi-runner-hidden.js`，登記時使用展開後的絕對路徑。三行原生 JScript 透過 `WScript.Shell.Run(command, 0, true)` 啟動原 `start-hapi-runner.ps1`，建立時隱藏、等待完成並回傳退出碼；不依賴 VBScript 或新服務套件。

原 PowerShell 內容、工作目錄、登入與一分鐘 triggers、principal、IgnoreNew、復原 settings 均保留，修改前後 XML 比較一致。每分鐘 trigger 仍需保留，可接回升級 handoff 後離開排程父子關係的 Runner。

## 實證與邊界

- wrapper 探針確認等待三秒並回傳指定退出碼 37。
- 第一次實際停止 Runner 後，58.4 秒內由排程恢复；六個其他基準程序身分不變，同一 canary session 復原後仍完成模型回覆，測試 session 隨後封存。
- 第二次獨立復原監看 85 秒，程序事件與桌面視窗枚舉均無 WindowsTerminal/OpenConsole 啟動或 HAPI 可見終端機視窗。隱藏 conhost 本身不代表閃窗；不要混入使用者瀏覽器或新建 session 的視窗。
- 當次 Runner 版本為 0.29.0.6，最終心跳與 Hub active 正常；未做重開機或登出測試。版本與程序狀態是歷史實證，未來操作需重查。
- 復原測試先比對 PID 與 `runner start-sync` 身分，再用 Runner 本機停止端點；避免不加辨識地執行可能落到 Windows `taskkill /T` 的停止流程，波及既有 sessions。

原 task XML 備份為 `<backup-dir>/pre-hidden-task.xml`；可用 `Register-ScheduledTask` 還原原任務，但舊 Action 也會帶回閃窗。保留原 PowerShell 與備份。操作文件與可部署腳本位於 HAPI skill 的 `references/config.md`、gist mirrors 與 `scripts/start-hapi-runner-hidden.js`；shared-skills PR 33、transfer_MAC PR 44 已合併，CI 與 review 通過。
