---
title: swop 上的 gemini-link：經 SSH 轉給 zeus 執行
scope: machines/swop
machine: swop
status: active
updated: 2026-10-24
tags:
  - gemini-link
  - windows
  - ssh
---

swop（Windows，Git Bash）原本沒有 `gemini-link`。transfer_MAC 的官方 client（`stow/core/.local/bin/gemini-link`）在 Windows 原生 Python 下無法使用，因為它要求 config 為 `0600`，而 Windows 的 `stat` 一律回報 0666。WSL 的 Ubuntu 雖已安裝但無法啟動：Virtual Machine Platform／Hyper-V 沒開，屬系統設定，不由 agent 變更。

目前的做法：`~/.local/bin/gemini-link` 是 sh wrapper，先在本機驗證參數（只允許 `swear01/[A-Za-z0-9._-]+`），再 `ssh -o BatchMode=yes swear01@140.112.171.138`（zeus，swear01 NAS 共用 home，config 權限 0600）執行 `~/.local/bin/gemini-link`。swop 不保存 token，transfer_MAC 也不需改碼。zeus 的 client 與 transfer_MAC 那份去掉 CRLF 後的 SHA-256 相同。

2026-10-24 驗證：`gemini-link swear01/transfer_MAC` 回傳 `already-linked`；新 repo `swear01/idle-spiral-openspeedy-patch` 回傳 `linked`；錯誤的 owner 與名稱都以 exit 2 拒絕。`~/.local/bin` 只在 Git Bash 的 PATH 內，不在 Windows PATH；PowerShell 要用 bash 呼叫。zeus 連不到時，linking 會失敗，依規則必須停止 push。
