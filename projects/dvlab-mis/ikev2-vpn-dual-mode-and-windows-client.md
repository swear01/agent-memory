---
title: DVLab IKEv2 VPN 現況與 Windows 用戶端排查
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-28
tags: [vpn, ikev2, windows, powershell, eap, psk, dvlab-mis]
---

# DVLab IKEv2 VPN 現況與 Windows 用戶端排查

## 架構現況

- 2026-09-21 使用者確認 Windows 現在也使用 PSK，並回報 Windows 與 Mac 同時連線會互踢；根因尚未驗證。
- Windows PSK 的用戶端名稱及 Local ID 尚待核對；防火牆 PSK 端點為 `.141`。不可再把舊 EAP 獨立端點紀錄當作現況，或據此排除 Windows/Mac 連線衝突。
- 本地維運文件與 `<remote-home>/dvlab-vpn/latest/DVLab-IKEv2-VPN.zip` 在查核時仍記載／包含 2026-07-26 EAP 部署。`latest` 路徑和記憶更新日期均不足以證明內容反映目前部署，必須核對實際用戶端與資料來源。
- 以下 EAP 與腳本排查為歷史紀錄，不是目前 PSK 的安裝指引。

## Windows 腳本 PropertyNotFoundStrict 根因與處置

- **現象**：同學執行 `Install-DVLab-IKEv2.ps1` 時中斷，報錯 `Set-RasphonePreSharedKey : 在此物件上找不到屬性 'Count'。請確認該屬性存在。`（`PropertyNotFoundStrict`）。
- **根因**：
  1. 該同學使用的是 2026-07-17 的早期安裝包。早期腳本在 Windows 上嘗試寫入 PSK，函式 `Set-RasphonePreSharedKey` 包含 `$lines = Get-Content -LiteralPath $PbkPath -Encoding Unicode`。
  2. Windows 原生產生的 `rasphone.pbk` 常為 ANSI 或 UTF-8。強制以 Unicode 讀取會使換行無法識別，回傳單一 scalar 字串（或空白檔案為 null）。
  3. 腳本開頭設有 `Set-StrictMode -Version Latest`，存取 scalar 物件的 `.Count` 會直接觸發例外終止。
- **處置**：
  - 當時處置為改用 EAP 安裝包，不再呼叫 `Set-RasphonePreSharedKey`；Windows 後續已改用 PSK，勿將此歷史處置當作現行安裝指引。
  - 若需維護舊版或撰寫類似 PowerShell 腳本，檔案讀取行數應一律使用強制陣列 `@(Get-Content -LiteralPath $PbkPath)`，並動態偵測檔案 BOM / 編碼；修改電話簿等次要設定需加 `try-catch` 防禦，避免阻斷主體 VPN profile 建立。

## 文件與公開說明邊界

- 公開說明文件保留給成員閱讀之安裝指引，不直接列出明文認證帳密，引導成員透過驗證帳號由內部伺服器取得包含設定與認證資訊的安裝包。
- 排查認證方式時，先核對實際用戶端與 profile，不能只憑作業系統推斷 Windows 必用 EAP 帳密。

## PSK 同時連線與客戶端識別（2026-09-21 查核）

- 本地維運文件位於 `<project-root>/playground/dvlab-mis-docs/IKEv2 Remote Access(private).md`；內容含機密，查詢時只輸出需要的非機密欄位。
- 文件記載 PSK gateway 的 `Peer ID Type` 為 `Any`、connection 為 `Remote Access (Server Role)`。這是文件紀錄，不是即時防火牆查核；不得據此宣稱已排除現行限制。
- 正式 ZIP 內 Apple IKEv2 payload 未設定 `LocalIdentifier`。未設定不能直接推斷所有裝置送出相同 ID，須檢查實際 IKE IDi 或設備日誌。
- 本地 Android 說明將 `IPSec identifier` 標成 Remote ID 並要求填伺服器位址。AOSP `Ikev2VpnProfile.java` 的 `fromVpnProfile()` 將 `profile.ipsecIdentifier` 傳入 Builder 的 user identity，`toVpnProfile()` 也將 `getUserIdentity()` 存入該欄位。因此不能把 Android 原生此欄位當成 Apple RemoteIdentifier；照同一值設定會共用客戶端身分，值得優先排查。
- Zyxel Community 的 `Multiple IKEv2 gateways in parallel`（discussion 16173）有 FLEX200 使用者回報相同 client Local ID 導致後連線替換前連線；員工確認不同 client Local ID 可同時連線。該案例使用憑證且涉及多 gateway，只能支持排查方向，不足以證明本部署 PSK 互踢的根因。
- 尚未做雙裝置同時連線驗證，也未修改正式設定或安裝包。

## 防火牆唯讀實查（2026-09-21）

- 從 zeus 使用維運文件中的 LAN 管理位址、既有 known_hosts 與管理認證，可 SSH 登入 USG FLEX 200；WAN 位址沒有 known_hosts 並不表示沒有可用管理入口。
- 此設備的 SSH 遠端 command 模式回報 `% session is not found`；`ssh -tt` 進入互動會話並由 stdin 傳入唯讀命令成功。不得因此跳過主機金鑰驗證。
- 已驗證可用：`show version`、`show running-config`、`show isakmp sa`、`show sa monitor`、`show logging entries category ike`、`show logging debug entries category ike`。`show ikev2 ?` 與 `show vpn ?` 在此會話回報 parse error，不要當作有效查詢。
- 韌體 V5.42(ABUI.1)。現行 `RemoteAccess_IKEv2` 為 pre-share、`peer-id type any`、remote-access-server；位址池有 250 個位址。設定未見每把 PSK 單連線限制，但這不足以排除實作的重複身分／來源位址處理。
- 2026-09-21 查核時 EAP gateway 仍存在且啟用；這是當時狀態，不代表 Windows 正在使用它。此次初次即時檢查只見一條 PSK IKE SA；尚不能從伺服器輸出辨認其作業系統。
- 現存 IKE 日誌只有近期保活，未涵蓋使用者回報的互踢事件。尚未取得雙裝置 IDi 或刪除原因，不可宣稱 Local ID 衝突已證實。下一步是兩台受控重現時同步讀取 IKE 日誌。

## EAP 退役與 utux 位址（2026-09-27）

- 使用者確認 Windows 現用 PSK 後，授權停用舊 EAP 並將原 `.143` 分配給 utux。USG FLEX 200 已將 `RemoteAccess_IKEv2_EAP` 的 IKEv2 policy 和 crypto map 設為 deactivate；即時讀回 EAP `active: no`，PSK `RemoteAccess_IKEv2` 仍為 `active: yes`。
- `wan2:1` 保留 `.143`，描述改為 utux；新 1:1 NAT 對應 utux 的既有內網位址。`.142:25565` port forward 曾在 2026-09-27 刪除，但 2026-09-28 即時讀回已恢復並啟用，亦轉送 utux。WAN→utux 的既有政策只允許 Minecraft TCP 25565；內網 NAT loopback 下掃到 SSH 開放，不能據此推斷 WAN 的 SSH 也開放。
- 2026-09-28 在實驗室內測到 `.143:25565` 和 `.142:25565` 均可連；外部視角尚未驗證。防火牆 `write` 無錯誤，但設備不支援 `show startup-config`，未直接讀回持久化設定。本機網路文件已按即時設定修正；Google Drive 正式文件須另外讀回驗證。
- 2026-09-28 HAPI `inspect-peer --limit 100` 的 `messages` 只擷取 Hub 訊息列中 CLI 可辨識的文字；對已封存的 Pi 工作階段只顯示三則 user 文字，沒有可讀的 Pi agent 文字。這不能區分 Hub 未儲存、同步中斷或擷取器未解析，也不能推論 Pi 在 Mac 未顯示或執行，或判定 Google 文件寫入狀態；須以正式 Google 文件讀回為準。
- 2026-09-28 以共用 DVLab 帳號直接使用 Docs API 增量更新既有 `IKEv2 Remote Access(private)` 與 `353 network setup manual(private)`：Windows 現用 `.141` PSK、舊 `.143` EAP policy／crypto map 停用且 `.143` 分配 utux；舊 EAP 安裝腳本標為歷史資料。兩份原生 Google 文件均以 Docs 讀回、Drive `text/plain` 匯出及 owner-only Restricted 權限檢查確認；網路文件的內部 VPN 連結改標 Google Docs，原連結仍指向 Google 文件。

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
