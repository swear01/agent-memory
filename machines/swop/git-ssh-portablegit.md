---
title: swop：git 經 Windows OpenSSH 9.5p2 會卡住，skills repo 改用 PortableGit 的 ssh
scope: machines/swop
machine: swop
status: resolved
created: 2026-10-25
updated: 2026-10-26
tags:
  - git
  - ssh
  - windows
  - skillshare
---

# swop 的 git SSH 卡住

## 症狀

swop 的 Git for Windows 2.49.0 安裝時選了 `SSH Option: ExternalOpenSSH`，沒有自帶 ssh，git 會呼叫系統的 Win32 OpenSSH `OpenSSH_for_Windows_9.5p2`。`skillshare pull`（即 skills repo 的 `git pull`）卡住超過 30 分鐘不結束。

## 已驗證

- 單獨執行 `ssh -T git@github.com` 會成功；stdin 換成檔案或 pipe 也成功，port 22 與 `ssh.github.com:443` 都可以。
- 只要由 git 呼叫同一個 ssh.exe，認證與 exec 完成後就收不到任何封包。protocol v0 或 v2、經過或不經過 `sh.exe` 結果都一樣。
- 用 ssh alias 改走 443 沒有幫助，已還原。
- 改用官方 PortableGit 2.49.0 內附的 MSYS `OpenSSH_9.9p2` 後，同一把金鑰、同一個 repo 立即正常。

根因沒有查明。推測與 git 啟動 Win32 OpenSSH、接上 pipe 的方式有關，未驗證。

## 目前設定

- PortableGit 解壓在 `%LOCALAPPDATA%\PortableGit-skillshare`。安裝檔是官方 `PortableGit-2.49.0-64-bit.7z.exe`，Authenticode 簽章有效（Johannes Schindelin），SHA-256 `BC980A64E875304EA5AA88386FDA37E8A0089D0F2023616B9995B1CA75B471DD`。
- 只改了 skills repo（`%APPDATA%\skillshare\skills`）的 `core.sshCommand`：
  `C:/Users/<user>/AppData/Local/PortableGit-skillshare/usr/bin/ssh.exe -i C:/Users/<user>/.ssh/id_ed25519_skillshare -o IdentitiesOnly=yes`
- 已安裝的 Git、全域 git 設定與 remote 都沒動。
- 還原方式：把 `core.sshCommand` 改回 `ssh -i ... -o IdentitiesOnly=yes`，再刪除該資料夾。

## 2026-10-26 已解決

用 `winget upgrade --id Git.Git --custom "/o:SSHOption=OpenSSH"` 把 Git for Windows 從 2.49.0 升到 2.56.0.3，同時把 SSH Option 從 `ExternalOpenSSH` 改成內附 OpenSSH，git 現在用 `C:\Program Files\Git\usr\bin\ssh.exe`（MSYS `OpenSSH_10.6p1`）。系統的 Win32 OpenSSH 9.5p2 沒有動，swop 的 sshd 仍然靠它。

skills repo 的 `core.sshCommand` 已改回 `ssh -i C:/Users/<user>/.ssh/id_ed25519_skillshare -o IdentitiesOnly=yes`。`git fetch` 和 `skillshare pull` 都能正常結束，HEAD 和 GitHub 一致。2026-10-26 確認沒有任何設定、PATH 或程序引用後，已刪除 `%LOCALAPPDATA%\PortableGit-skillshare`（約 402 MB）。若要再用這個 workaround，需重新下載上面同一個官方安裝檔並核對 SHA-256。

注意：Git installer 升級時會沿用上一次選的選項，要改 SSH Option 必須明確傳 `/o:SSHOption=OpenSSH`。

## 遠端操作 swop 的注意事項

- ssh 預設 shell 是 cmd，PowerShell 要用 `-EncodedCommand`（UTF-16LE base64）。
- PowerShell 經 ssh 的輸出可能變成 CLIXML 而遺失。比較可靠的做法是 scp `.ps1` 上去，讓腳本把輸出寫到 `%TEMP%` 的 log，再用 `type` 讀回。
- git 網路卡住時可以離線更新：在 Mac 做 `git bundle`，scp 到 swop，fetch 到 `refs/remotes/origin/main` 後 `merge --ff-only`，再 `skillshare sync`。

相關：[[fleet-cli-and-skill-upgrade]]、[[ssh-443-bounded-fallback]]
