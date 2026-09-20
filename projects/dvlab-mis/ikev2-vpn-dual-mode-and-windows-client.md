---
title: DVLab IKEv2 VPN 雙軌架構與 Windows 用戶端排查
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-21
tags: [vpn, ikev2, windows, powershell, eap, psk, dvlab-mis]
---

# DVLab IKEv2 VPN 雙軌架構與 Windows 用戶端排查

## 架構現況

- 實驗室 IKEv2 VPN 採平台雙軌制：
  - Apple / Android：端點走純 PSK（Pre-Shared Key）模式，不需個人或共用使用者帳號密碼。
  - Windows 10 / 11：現行版本為 `DVLab IKEv2 EAP`，走獨立端點與 EAP-MSCHAPv2 認證，需匯入專用 CA 根憑證並輸入共用認證帳密。
- 正式安裝包位於伺服器 `<remote-home>/dvlab-vpn/latest/DVLab-IKEv2-VPN.zip`，由發布流程維護。排查連線問題時，應先確認用戶端取得的是最新釋出包，不可單憑本機暫存目錄的歷史 ZIP 作為依據。

## Windows 腳本 PropertyNotFoundStrict 根因與處置

- **現象**：同學執行 `Install-DVLab-IKEv2.ps1` 時中斷，報錯 `Set-RasphonePreSharedKey : 在此物件上找不到屬性 'Count'。請確認該屬性存在。`（`PropertyNotFoundStrict`）。
- **根因**：
  1. 該同學使用的是 2026-07-17 的早期安裝包。早期腳本在 Windows 上嘗試寫入 PSK，函式 `Set-RasphonePreSharedKey` 包含 `$lines = Get-Content -LiteralPath $PbkPath -Encoding Unicode`。
  2. Windows 原生產生的 `rasphone.pbk` 常為 ANSI 或 UTF-8。強制以 Unicode 讀取會使換行無法識別，回傳單一 scalar 字串（或空白檔案為 null）。
  3. 腳本開頭設有 `Set-StrictMode -Version Latest`，存取 scalar 物件的 `.Count` 會直接觸發例外終止。
- **處置**：
  - 引導同學直接自伺服器重新取得最新正式安裝包（已切換至 EAP 模式，不再呼叫 `Set-RasphonePreSharedKey`）。
  - 若需維護舊版或撰寫類似 PowerShell 腳本，檔案讀取行數應一律使用強制陣列 `@(Get-Content -LiteralPath $PbkPath)`，並動態偵測檔案 BOM / 編碼；修改電話簿等次要設定需加 `try-catch` 防禦，避免阻斷主體 VPN profile 建立。

## 文件與公開說明邊界

- 公開說明文件保留給成員閱讀之安裝指引，不直接列出明文認證帳密，引導成員透過驗證帳號由內部伺服器取得包含設定與認證資訊的安裝包。
- 排查同學詢問「是否有帳號密碼」時，需先區分作業系統：Apple/Android 無帳密，Windows 需填入安裝包隨附之共用認證。
## Windows IPsec 300 秒閒置斷線：已驗證設定與未完成驗證

2026-09-21 在 Swear01_PC 的 Windows 原生 IKEv2 EAP 連線觀察到兩次相同序列：建立 IPsec Quick Mode SA 後整整 300 秒結束，再約 2 秒由 RasClient 記錄 20226、reason code 828（ERROR_IDLE_TIMEOUT）。同期檢查未見 Wi-Fi 斷線或休眠事件；這支持閒置回收假說，但不代表已排除所有其他原因。

- VPN profile 的 `IdleDisconnectSeconds=0`，不等於底層 IPsec SA 永不閒置回收。
- 提升管理員權限後，`Get-NetIPsecQuickModeSA` 讀到入站與出站 `IdleDurationSeconds=300`、`LifetimeSeconds=3600`；Main Mode lifetime 為 28800 秒。應區分 idle timeout 與密鑰 lifetime。
- `Get-NetFirewallSetting -PolicyStore ActiveStore` 原為 `MaxSAIdleTimeSeconds=300`；PersistentStore 原為 `NotConfigured`。
- 經使用者要求，執行 `Set-NetFirewallSetting -PolicyStore PersistentStore -MaxSAIdleTimeSeconds 3600`。讀回 PersistentStore、ActiveStore 均為 3600；此設定影響全局 IPsec，不是單一 VPN profile。
- **當時驗證範圍**：修改後既有入站／出站 SA 仍顯示 300 秒；後續已完成重連與服務重啟驗證，結果仍為 300 秒，詳見下方。尚未驗證一小時閒置存活，不能宣稱 VPN 斷線已修復。
- 微軟 `Set-NetFirewallSetting` 的 MaxSAIdleTimeSeconds 文件列出支援範圍 300–3600 秒，沒有列出「永不」；不要把此參數設 0 當成已確認的關閉方式，也不要把延長時間描述為永久保活。
- 非提升權限下讀 Quick Mode SA 會 Access is denied；此次 `netsh advfirewall show global` 即使提升權限仍回報 0x2，但 PowerShell 的 SA 與全局設定查詢成功。不要僅靠 netsh 失敗判定資料無法取得。

### 同日重連後追查

- 後續管理員唯讀檢查確認：重新建立的 IKEv2 入站與出站 SA 仍為 `IdleDurationSeconds=300`，而 ActiveStore 與 PersistentStore 的 `MaxSAIdleTimeSeconds` 都是 3600。因此不能再把差異僅歸因於修改前既有 SA；全局設定寫入不代表 RAS IKEv2 通道採用。
- 一次連線持續約 28 分鐘後，事件依序為 Quick Mode SA ended、約兩秒後 Main Mode terminated 與 RasClient 828。828 是閒置逾時，不能解讀成每次連線總壽命固定五分鐘；未取得最後資料封包時間，不能宣稱已量測該次精確閒置區間。
- 本機 `HKLM\SYSTEM\CurrentControlSet\Services\RemoteAccess\Parameters\Ikev2` 的 `idleTimeout` 當時為 300；RemoteAccess 服務為 Disabled/Stopped，RasMan 與 IKEEXT 為 Running。Microsoft MS-RRASM 的 Other Miscellaneous Configuration Information 有此值的 RRAS 說明，但不能僅憑數值相同就宣稱根因已確定；後續修改測試見下方。
- 後續已完成服務重啟；系統重開及閒置／流量對照實驗仍未進行，不可保證重開機能解決。

### 同日用戶授權修改與服務重啟結果

- Windows 11 Home build 26200 上，提升權限執行 `netsh ras set ikev2connection idletimeout=60` 回報 `The parameter is incorrect.`；加上原值 `nwoutagetime=30` 仍失敗，登錄值未變。不要把官方文件列出語法視為本機成功證據。
- 將上述 IKEv2 登錄值 `idleTimeout` 由 300 改為 3600 後，`netsh ras show ikev2connection` 顯示 60 分鐘；但以 `rasdial` 斷線重連後，新入站與出站 SA 仍為 `IdleDurationSeconds=300`。
- 隨後斷開 VPN，成功重啟 RasMan 與 IKEEXT，再成功重連；又一組新 SA 的 idle 值仍為 300。這排除了單純重連或重啟這兩個服務即可套用的假說；尚未驗證系統重開、其他快取或保活方案。
- 修改後 IKEv2 登錄設定保留 3600，全局設定仍為 3600，實際 SA 仍為 300；VPN 已恢復 Connected。不可宣稱五分鐘問題修好。
