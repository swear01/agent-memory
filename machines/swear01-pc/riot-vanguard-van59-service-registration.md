---
title: SWear01_PC 的 Riot Vanguard VAN 59 服務註冊失配
scope: machine
machine: SWear01_PC
status: active
created: 2026-09-04
updated: 2026-09-04
tags:
  - windows
  - riot-vanguard
  - league-of-legends
  - van-59
  - service-control-manager
---

# 已驗證狀態

- Windows 11 build 26200、League of Legends 16.17 與 Riot Vanguard 1.19.0.6 發生 VAN 59。
- `sc queryex vgc` 顯示 `WIN32_EXIT_CODE 1`（Windows 的 `ERROR_INVALID_FUNCTION`／`Incorrect function`），但更具體的 `SERVICE_EXIT_CODE` 是 `59`。League Client 同時記錄 `VGC connect: Failed to connect to Vanguard client`、Vanguard status `-81`、exit code `59`；因此 Event ID 7023 的 `Incorrect function` 是表層翻譯，不是獨立根因。
- `vgc.exe` 與 `vgk.sys` 均存在於標準 Vanguard 安裝目錄，版本一致且 Authenticode 簽章有效；Windows Code Integrity 紀錄與 Defender 偵測沒有 Vanguard 封鎖事件。
- `vgk` 不存在於 Service Control Manager、`Win32_SystemDriver` 或 `HKLM\SYSTEM\CurrentControlSet\Services`，但 `vgk.sys` 檔案仍存在。`vgc` 則為 `DEMAND_START`。
- 本機沒有啟用 Vanguard Pre-Check：VGTray settings 的 Pre-Check 欄位均為 `0`，Riot Client 的 `onDemandVanguard.enabled` 為 `false`。依 Riot 官方文件，未啟用 Pre-Check 時 `vgc` 應設為 Automatic；所以目前的 `vgc` 啟動類型與缺失的 `vgk` 註冊，構成可重現的 Vanguard 安裝／服務註冊失配。
- Secure Boot 當時為關閉、TPM 2.0 由 League Client 報告為開啟、VBS/HVCI 未運行。這可能在修復 Vanguard 後導致另一個明確的 security/restriction code，但官方 VAN 59 流程沒有把它列為直接成因，而且目前沒有 Code Integrity 封鎖證據。

# 診斷邊界與後續驗證

- 不要只根據 Event ID 7023 或手動啟動 `vgc` 的對話框判斷；同時查看 `sc queryex vgc` 的 service-specific exit code、`sc query vgk`、Pre-Check 狀態及 League Client 的 Vanguard connect 訊息。
- 目前可確定失敗發生在本機 League Client 與 Vanguard client/driver 的連接層，不是一般 Riot 登入或 Internet 連線失敗。Vanguard 自身 `.log` 是受保護的二進位格式，精確內部子原因只能由 Riot 的診斷工具／客服解碼。
- 官方 VAN 59 順序是：完整重啟 Riot Client、移除並重裝 Vanguard、再重裝遊戲。此機單純把 `vgc` 改成 Automatic 不足以補回缺失的 `vgk` 註冊，也不應手工猜測並建立 `vgk` registry/service 參數。
- 尚未執行修復。重裝 Vanguard 並重開機後，必須重新驗證 `vgc` 啟動類型、`vgk` 是否註冊／載入、兩個服務狀態、League Client 是否能連上 Vanguard，以及 VAN 59 是否在自訂或訓練模式中消失；未完成這些檢查前不可把重裝視為已修復。
