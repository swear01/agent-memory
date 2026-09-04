---
title: SWear01_PC 的 Riot Vanguard VAN 59／VAN 57 服務註冊與開機載入失配
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
  - van-57
  - service-control-manager
---

# 已驗證狀態

- Windows 11 build 26200、League of Legends 16.17 與 Riot Vanguard 1.19.0.6 發生 VAN 59。
- `sc queryex vgc` 顯示 `WIN32_EXIT_CODE 1`（Windows 的 `ERROR_INVALID_FUNCTION`／`Incorrect function`），但更具體的 `SERVICE_EXIT_CODE` 是 `59`。League Client 同時記錄 `VGC connect: Failed to connect to Vanguard client`、Vanguard status `-81`、exit code `59`；因此 Event ID 7023 的 `Incorrect function` 是表層翻譯，不是獨立根因。
- `vgc.exe` 與 `vgk.sys` 均存在於標準 Vanguard 安裝目錄，版本一致且 Authenticode 簽章有效；Windows Code Integrity 紀錄與 Defender 偵測沒有 Vanguard 封鎖事件。
- `vgk` 不存在於 Service Control Manager、`Win32_SystemDriver` 或 `HKLM\SYSTEM\CurrentControlSet\Services`，但 `vgk.sys` 檔案仍存在。`vgc` 則為 `DEMAND_START`。
- 本機沒有啟用 Vanguard Pre-Check：VGTray settings 的 Pre-Check 欄位均為 `0`，Riot Client 的 `onDemandVanguard.enabled` 為 `false`。依 Riot 官方文件，未啟用 Pre-Check 時 `vgc` 應設為 Automatic；所以目前的 `vgc` 啟動類型與缺失的 `vgk` 註冊，構成可重現的 Vanguard 安裝／服務註冊失配。
- Secure Boot 當時為關閉、TPM 2.0 由 League Client 報告為開啟、VBS/HVCI 未運行。這可能在修復 Vanguard 後導致另一個明確的 security/restriction code，但官方 VAN 59 流程沒有把它列為直接成因，而且目前沒有 Code Integrity 封鎖證據。

# 已驗證修復

- 使用 Vanguard 官方解除安裝程式完整移除後重開機，再由 Riot Client 重裝，成功補回 `vgk` 服務；`vgk` 為 `SYSTEM_START`，`vgc` 設為 `AUTO_START`。
- 第一次重開機發生在 Vanguard 重裝之前，因此重裝後雖可手動啟動 `vgk` 與 `vgc`，`vgk` 並未經過系統開機階段載入。League Client 可短暫完成 `VGC connect+login OK`，隨即使 `vgc` 以 service-specific exit code `57` 結束；Riot Client 收到 `PLAYER_LACKS_VANGUARD_SESSION`，玩家會被移出列隊。
- 將 `C:\Windows\vgkbootstatus.dat` 改名保留備份，確認 `vgc` 為 Automatic，然後再次完整重開機。新開機後 `vgk`、`vgc` 均持續 Running、exit code `0`，並建立新的 `vgkbootstatus.dat`。
- 修復後 League Client 依序記錄 `VGC connect`、`VGC login`、`VGC connect+login OK` 與 `Successfully logged in to Vanguard client`；超過一分鐘監看未再發生 7023、exit code 57 或 Vanguard disconnect，遊戲程序也成功啟動。這確認真正有效的步驟是讓新安裝的 `vgk` 經過完整開機載入並重建 boot status，而不是只手動啟動服務。
- 使用者後續確認修復完成；此結果與服務、League Client 及遊戲程序的即時驗證一致。
- 排除項目：`DevOverrideEnable` 不存在、Driver Verifier 未啟用、BCD 無 `testsigning`／`nointegritychecks`／`flightsigning`、Winsock 無 Bonjour `mdnsNSP.dll`、Defender 與 Code Integrity 無 Vanguard 封鎖事件。Radmin VPN 與 ZeroTier 暫停後未執行 League 連線測試，且恢復後經正確開機仍修復，因此沒有證據認定它們是根因。
- Secure Boot 仍為關閉，TPM 2.0 已啟用；本次 VAN 57 已在 Secure Boot 關閉的狀態下修復，因此 Secure Boot 不是此事件的直接原因。

# 診斷邊界與後續驗證

- 不要只根據 Event ID 7023 或手動啟動 `vgc` 的對話框判斷；同時查看 `sc queryex vgc` 的 service-specific exit code、`sc query vgk`、Pre-Check 狀態及 League Client 的 Vanguard connect 訊息。
- 目前可確定失敗發生在本機 League Client 與 Vanguard client/driver 的連接層，不是一般 Riot 登入或 Internet 連線失敗。Vanguard 自身 `.log` 是受保護的二進位格式，精確內部子原因只能由 Riot 的診斷工具／客服解碼。
- 官方 VAN 59 順序是：完整重啟 Riot Client、移除並重裝 Vanguard、再重裝遊戲。此機單純把 `vgc` 改成 Automatic 不足以補回缺失的 `vgk` 註冊，也不應手工猜測並建立 `vgk` registry/service 參數。
- Vanguard 重裝發生在某次 Windows 開機之後時，即使安裝器可即時啟動 `vgk`，仍應再完整重開機，並以 League Client 的 `Successfully logged in to Vanguard client` 與服務持續 Running 作為成功標準。
