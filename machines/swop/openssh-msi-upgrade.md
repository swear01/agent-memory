---
title: swop：系統 OpenSSH 9.5p2 改用 Win32-OpenSSH MSI（現為 10.4p1）；MSI 防火牆規則只開 Private
scope: machines/swop
machine: swop
status: active
confidence: high
created: 2026-10-26
updated: 2026-10-26
tags:
  - windows
  - openssh
  - sshd
  - firewall
  - msi
source_refs:
  - live:swop-2026-10-26
  - github:PowerShell/openssh-portable/contrib/win32/install/server.wxs
---

# 為什麼要裝 MSI

Windows 內建（Optional Feature）OpenSSH 只隨功能更新變動。swop 已是 Windows 11 26H2 build 26300，內建仍是 `OpenSSH_for_Windows_9.5p2`。較新的版本只有 PowerShell/Win32-OpenSSH 的 GitHub release（全部標 Preview）。winget `Microsoft.OpenSSH.Preview` 2026-10-26 時是 10.0.0.0；GitHub 已有 10.4.0.0p1（2026-10-20）。

# 遠端安裝會斷線的原因（已實測）

MSI 會把 `sshd` 服務改指 `C:\Program Files\OpenSSH\sshd.exe`，並新增規則 `OpenSSH SSH Server Preview (sshd)`：只放行該程式、`Profile=private`（見 `server.wxs` 的 `firewall:FirewallException`）。內建規則 `OpenSSH-Server-In-TCP` 雖是 Any profile，卻綁 `%SystemRoot%\system32\OpenSSH\sshd.exe`。swop 的 Wi-Fi（TP-LINK-LISA）與 ZeroTier 是 **Public**，所以第一次安裝後所有新連線 timeout（不是 refused）。

解法：安裝後另建 `OpenSSH-Server-In-TCP-PF`，Program 指向 Program Files 的 sshd.exe、`-Profile Any`、TCP 22。

# 目前狀態

- sshd 與 ssh client：2026-10-26 再從 GitHub 升到 `10.4.0.0p1-Preview`（`OpenSSH_for_Windows_10.4p1`、LibreSSL 4.3.3），Auto、Running、`C:\Program Files\OpenSSH\sshd.exe`。Mac 連線的 kex 變成 `mlkem768x25519-sha256`，post-quantum 警告消失。host keys、`sshd_config`、`administrators_authorized_keys` 沿用 `C:\ProgramData\ssh`，known_hosts 不用改。
- ssh-agent：MSI 會設成 Auto，已改回原本的 Disabled。
- Machine PATH：`C:\Program Files\OpenSSH\` 排在 `System32\OpenSSH\` 前，`ssh` 解析到新版。
- 內建 capability 沒移除，System32 的 9.5p2 檔案仍在。
- Git for Windows 用自己內附的 ssh，不受影響（見 [[git-ssh-portablegit]]）。

# 安全做法與回退

安裝放在 SYSTEM scheduled task 跑，不依賴當下 SSH 連線；另設 15 分鐘後的 SYSTEM rollback task，沒看到 `C:\ProgramData\OpenSSH-upgrade\OK` 就自動回退。第一次失敗即由它在 6 秒內救回。PowerShell 變數不分大小寫，`$W` 與 `$w` 會互相覆蓋。

備份與腳本留在 `C:\ProgramData\OpenSSH-upgrade`：`ssh-backup`、`ssh-acl.txt`、`sshd-qc.txt`、`sshd-privs.txt`、`machine-path.txt`、`openssh.msi`、`rollback.ps1`。回內建版的腳本已改名 `rollback-inbox.ps1`；它會 `msiexec /x`、刪 PF 規則、重建指向 System32 的 sshd/ssh-agent 服務、還原設定與 ACL、PATH。

# MSI 之間升級（10.0 → 10.4 已實測）

MSI 用 `MajorUpgrade`，裝新版會先移除舊版再重建服務與 Private-only 規則；自建的 PF 規則不受影響，但仍由 `post.ps1` 重建。MSI 不能直接降版，所以回退目標是「`/x` 目前版本，再 `/i` 上一版 MSI」，失敗再接 `rollback-inbox.ps1 -Force` 回內建 9.5。

`C:\ProgramData\OpenSSH-upgrade` 內：`openssh-10.4.msi`（SHA-256 `788D8643464A01EB63640A2FD746F9ED23D106B5A901A35F88DC8CD40E446AC9`，和 GitHub asset digest 相符）、`openssh-10.0.msi`、`post.ps1`（sshd Auto、ssh-agent Disabled、PF 規則、PATH 順序）、`rollback104.ps1`（回 10.0）、`rollback-inbox.ps1`（回內建），`v104\` 是升 10.4 前的設定/ACL/服務/PATH 備份。手動回 10.0：`rollback104.ps1 -Force`；回內建：`rollback-inbox.ps1 -Force`。

下次升級：下載新 MSI 並核對 GitHub digest 與 Authenticode，備份到新子資料夾，註冊 15 分鐘 rollback task，再用 SYSTEM task 安裝並跑 `post.ps1`，從 Mac 開新連線確認 remote software version 後才寫 OK、刪 task。

相關：[[hapi-terminal-flash]]
