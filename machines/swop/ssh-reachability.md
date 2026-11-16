---
title: swop SSH reachability: public IP flaps, Tailscale is the fallback
scope: machines/swop
machine: swop
status: active
confidence: high
created: 2026-11-17
updated: 2026-11-17
tags:
  - ssh
  - tailscale
  - windows
  - cursor-agent
source_refs:
  - live:swop-2026-11-17
---

# 連線路徑

- `ssh swop`（`stanl@118.150.242.12:22`，Host key 已在 `~/.ssh/config` alias）公網 SSH 會間歇性逾時（2026-09-28、2026-11-14 都逾時），2026-11-17 又恢復。fleet 筆記曾因此誤記成「未升級、未連 Hub」；逾時時先重試，再用 Tailscale。
- Tailscale 備援：主機名 `swear01-pc`，`stanl@100.69.9.16`，ping 可走區網 `192.168.0.236`。第一次連要接受 host key（`-o StrictHostKeyChecking=accept-new`）。
- SSH 預設 shell 是 cmd.exe；PowerShell 要用 `powershell -NoProfile -ExecutionPolicy Bypass -File <x>.ps1`。`irm ... | iex` 直接 pipe 在這裡會 `Access is denied`，改成先 `Invoke-WebRequest -OutFile` 再用 `-File` 跑。
- 2026-11-17 實測 swop HAPI 為 `0.35.0.1`、Runner 當天 01:25 已起來，不是未升級。

# Cursor Agent 更新

Windows 官方安裝器：`https://cursor.com/install?win32=true`（下載成 `.ps1` 再 `-File` 執行）。裝到 `%LOCALAPPDATA%\cursor-agent`，`cursor-agent` 與 `agent` 都在 PATH。2026-11-17 由 2026.09.26 升到 2026.11.13-eacade1。安裝器提示缺 Git for Windows 只影響 private worker。
