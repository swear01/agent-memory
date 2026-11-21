---
title: macOS LaunchAgent ad-hoc binary replacement must use a new inode
scope: machine
machine: swairM5
tags: [macOS, launchd, AMFI, code-signing, Swift]
status: active
updated: 2026-11-22
---

在 swairM5 上，直接用 `cp` 覆蓋一個仍由 LaunchAgent 執行中的 ad-hoc signed Mach-O，之後從原路徑啟動可能被 macOS AMFI 以 `SIGKILL (Code Signature Invalid)` 拒絕；`codesign --verify` 仍可能顯示 valid。實際診斷會出現 `cs_mtime=... != mtime=...` 與 `CODE SIGNING: rejecting invalid page`。

另外在 macOS 27.2.1（Darwin 27.2）上，`launchd` 會在 `bootstrap` 時把 ad-hoc binary 的 `cdhash` 鎖進該 job 的 `LWCR`（Launch Constraint）。就算已建立新 inode，單純執行 `launchctl kickstart -k gui/$UID/<label>` 仍會因為新舊 `cdhash` 不符而以 `OS_REASON_CODESIGNING | Launch Constraint Violation`（`Constraint not matched`）被 AMFI 擋掉。

部署這類 LaunchAgent binary 時：
1. 先在暫存路徑編譯、執行自測並用 `codesign --verify` 驗證，同時備份舊檔。
2. 刪除舊路徑、把已驗證的暫存檔移入形成新 inode。
3. 以 `launchctl bootout gui/$UID/<label>` 再 `launchctl bootstrap gui/$UID <plist>` 重新載入，讓 `launchd` 重新計算 `LWCR`，並確認 `launchctl print` 顯示 `state = running`。

本次（2026-11-22）以 `<home>/Library/Application Support/HAPI/lid-display-switch` 驗證：僅 `kickstart -k` 觸發 Launch Constraint Violation，改用 `bootout` + `bootstrap` 後立即正常啟動（`last exit code = (never exited)`）。
