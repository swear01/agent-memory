---
title: swop HAPI Runner 以 High integrity 執行與遠端提權流程
scope: machines/swop
machine: swop
tool: hapi/Windows-Task-Scheduler/PowerShell
status: active
created: 2026-10-30
updated: 2026-10-30
tags: [swop, hapi, runner, uac, schtasks, integrity-level, powershell, encodedcommand]
---

# 排程任務現況（2026-10-30）

swop 是 Windows 機器（使用者帳號 `stanl`），有兩個相關 scheduled task：

- `DeepSeek Gateway (SWOP)`：以 `SYSTEM` 執行，gateway 位於 `C:\ProgramData\DeepSeekGateway`，
  listen loopback `35001`。
- `HAPI Runner (SWOP)`：`wscript.exe //B //Nologo` 執行
  `<user-profile>\AppData\Local\Programs\Hapi\start-hapi-runner-hidden.js`。

依使用者要求，Runner task 已改成 `RunLevel Highest`，因此 swop 上的 HAPI session 預設以
High integrity（admin）執行。

# 陷阱：結束排程任務不會殺掉 daemon

`schtasks /End`／停止 Runner task **不會**結束已 daemonize 的 `hapi.exe`；舊的 Medium
integrity daemon 會繼續服務，看起來像設定沒生效。正確順序：

1. 先 kill `hapi.exe` process。
2. 再 `schtasks /Run /TN "HAPI Runner (SWOP)"`。
3. 驗證：開一個便宜的 HAPI session 執行 `whoami /groups`，確認出現 `High Mandatory Level`，
   驗完立即 archive 該 session，並檢查 hub 沒有殘留 session。

# 提權只能由使用者按 UAC

- HAPI 上的 Claude peer（及類似 agent）會拒絕對自己的 runner 提權或對自己觸發 UAC。
  這是合理的安全拒絕，**不要**換更寬鬆的 model 來繞過。
- 可行路徑：使用者把 Windows 機器註冊到 Grok Bot，由助理在該機逐條指令經使用者核准執行；
  需要提權時用
  `Start-Process powershell -Verb RunAs -Wait -ArgumentList '-NoProfile -ExecutionPolicy Bypass -EncodedCommand <b64>'`，
  由使用者親自點 UAC。

# 透過 bash 類 shell bridge 跑 PowerShell

經 bash-ish 的遠端 shell bridge 傳 PowerShell 時，`$` 變數（如 `$_`）會被展開／破壞。
改用 `powershell -NoProfile -EncodedCommand <b64>`，b64 為 UTF-16LE 編碼：

```bash
printf '%s' "$PS_SCRIPT" | iconv -f utf-8 -t utf-16le | base64 -w0
```
