---
title: macOS LaunchAgent ad-hoc binary replacement must use a new inode
scope: machine
machine: swairM5
tags: [macOS, launchd, AMFI, code-signing, Swift]
status: active
updated: 2026-08-22
---

在 swairM5 上，直接用 `cp` 覆蓋一個仍由 LaunchAgent 執行中的 ad-hoc signed Mach-O，之後從原路徑啟動可能被 macOS AMFI 以 `SIGKILL (Code Signature Invalid)` 拒絕；`codesign --verify` 仍可能顯示 valid。實際診斷會出現 `cs_mtime=... != mtime=...` 與 `CODE SIGNING: rejecting invalid page`。

部署這類 LaunchAgent binary 時，先編譯並驗證暫存檔，再備份舊檔；最後刪除舊路徑、把已驗證的暫存檔移入形成新 inode，完成後再 `launchctl kickstart -k gui/$UID/<label>`。本次以 `/Users/swear/Library/Application Support/HAPI/lid-display-switch` 驗證成功。
