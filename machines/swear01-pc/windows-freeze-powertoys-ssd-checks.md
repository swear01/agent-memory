---
title: Swear01_PC 死機後 PowerToys 與 SSD 檢查
scope: machine
machine: swear01-pc
status: active
updated: 2026-09-28
---

# 已完成的修復與檢查

- PowerToys 從 0.95.0 更新為 0.101.2362.0。依使用者決定停用 LightSwitch；保留 Awake 防休眠和 FancyZones。升級後全域功能開關與備份比對一致，模組設定檔雜湊一致；Awake/FancyZones 行程存在。
- 三個 SSD 的 C/D/E 磁碟區經 `Repair-Volume -Scan` 均為 NoErrorsFound，並核對 Chkdsk 26226 事件。這證明本次檔案系統掃描通過，不能排除硬體、韌體或核心資源問題。
- CrystalDiskInfo 檢查：Crucial T500 2TB 健康度 98%，NVMe media errors=0；Crucial P2 1TB 健康度 100%，media errors=0；Crucial MX500 500GB 剩餘壽命 51%，壞區與不可修正錯誤為 0。MX500 CRC 累計 2 筆、P2 Error Information entries 累計 4194，不能僅以歷史累計值指認死機原因，應觀察是否繼續增加。
- Crucial 官方韌體頁面在本機回傳 Request Rejected，無法確認適用最新版；未進行韌體更新。

## Windows 掃描方法的限制

本機提升權限的 PowerShell 直接啟動 `chkdsk.exe` 曾回報 Access denied，儘管其簽章及 RX 權限正常。相同提升權限下使用 `Repair-Volume -Scan` 完成非離線掃描，是本次驗證過的替代方式；不要為此修改系統檔 ACL 或複製執行檔繞過限制。

詳細輸出與設定備份保留在 `<user-home>/Documents/Codex/2026-09-28/no-more-older-messages-conversation-log` 的 outputs 與 work 目錄。原始 SMART 含裝置序號，WARP 設定可能含註冊資訊，勿直接放入共用記憶庫。Port 耗盡證據與監測設定見同目錄 `windows-port-exhaustion-investigation.md`。
