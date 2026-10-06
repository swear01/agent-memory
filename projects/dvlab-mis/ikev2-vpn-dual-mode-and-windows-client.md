---
title: DVLab IKEv2 VPN 現況與 Windows 用戶端排查
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-10-06
tags: [vpn, ikev2, windows, powershell, certificate, eap, psk, archive, dvlab-mis]
---

# DVLab IKEv2 VPN 現況與 Windows 用戶端排查

## 架構現況（2026-10-06，取代先前 Windows PSK 說明）

- Windows 原生 IKEv2 不能靠旧腳本寫入 PSK 欄位來使用 PSK 認證；「腳本能執行」並不是「VPN 認證成功」。現在改為 `MachineCertificate`，不需輸入 VPN 帳密。Apple / Android 原 PSK 保留。
- 非 H 系列 Zyxel USG FLEX 200、韌體 V5.42 上，兩种認證已可共用既有 `.141` 入口：原 PSK 使用 AES256/SHA256；新增 `DVLab_Windows_141` 的 Phase 1 和 Phase 2 都使用 AES128/SHA256，DH14、no PFS、獨立 Windows 位址池。新增連線加入 `IPSec_VPN` 及 `VPN_To_WAN_SNAT`；移除新增物件與群組成員後，running config 與之前逐位元組相同。`write` 成功，未獨立讀回 startup config。
- 網路出口遵循既有雙 WAN 路由，外部測試觀察到 `.141` 和 `.145`；連線入口 `.141` 不代表外網出口固定 `.141`。Windows 驗證腳本接受這兩個實驗室 WAN 位址。
- `.138:9443` 保留 HTTPS 自動簽發；`.138` 的 strongSwan 憑證 VPN 是已驗證備案，新包不使用它作為 VPN 入口。舊 `.143` EAP 維持停用。
- 以前記載的「Windows 使用 PSK」是舊安裝腳本／使用者回報，未證明 Windows 原生 IKEv2 PSK 成功。以下 September PSK 與 EAP 內容為歷史證據，不是現行操作指引。

## 同 IP 選錯規則的根因與補測

- 先前只更換 Phase 1 的 DH group，Phase 2 仍與 PSK 相同，Zyxel 選到 PSK 並回傳 PSK server AUTH。這些失敗不足以宣稱 Windows 必須換 IP。
- Zyxel 官方員工在 Community discussion「How to run two IKEv2 tunnels (full + split) on the same router?」、comment 79597 指出：不同 VPN 的 Phase 1 和 Phase 2 proposals 都應不同；相同組合可能依規則順序匹配錯誤規則。該官方案例不是完整相同的 PSK + Windows 組合，需本部署補測。
- 真正補測：原 PSK AES256/SHA256 不變，只在新增憑證規則及 Windows 候選腳本使用 AES128/SHA256（兩階段均不同）。外部 Oracle Linux strongSwan 成功進入 RSA 憑證規則，原 PSK 可同時連線；憑證中斷重連也成功。
- 一開始新位址池只能進內網、不能上網；將新增連線加入既有 `VPN_To_WAN_SNAT` 群組後，外網和 DNS 通過。不要只驗證 IKE SA 建立就宣稱 VPN 可用。

## 同一支 Windows 腳本自動簽發與驗證邊界

- 使用者要求成員首次安裝可在外網完成，不輸入邀請碼；同一支腳本內含共用 enrollment 密鑰。持有安裝包即能申請 VPN 憑證，僅向原成員範圍發放，不公開密鑰。這不是沒有信任依據的匿名安全簽發。
- `certreq` 在 LocalMachine 產生不可匯出的 RSA2048 私鑰及 CSR，送 HTTPS enrollment service 取得個別、有效一年、clientAuth 憑證。私鑰留在本機，不共用 PFX，不把 CA 私鑰放入 ZIP；有效憑證會重用，到期前重跑可換發。
- 服務位於 zeus `<remote-home>/.local/share/dvlab-windows-vpn/`，使用者 unit `dvlab-windows-enroll`；目錄 700、機密檔 600。簽發服務使用共用 client CA；Zyxel `.141` server certificate 是另一張 self-signed 憑證。安裝包內有兩張公開 `.crt`，脚本各自釘選 SHA256 並匯入 LocalMachine Root；client CA 用於 client issuer/EKU filter 和 HTTPS enrollment trust，server cert 用於信任 Zyxel。
- 外部 Linux 已驗證：HTTPS CA 信任、錯誤 token 403、無效 CSR 400、有效及重試 200；RSA IKEv2（無 IDr）、PSK 同時連線、內網 TCP、DNS 直接查詢、全流量外網和憑證重連。測試私鑰、token、PSK 暫存和 Oracle 臨時 VPN 套件已清理。
- Linux 測試端 resolvconf 無法自動寫入 system DNS；直接透過 VPN 向 DNS 查詢成功。這是客戶端測試限制，不能把 system DNS 自動設定宣稱通過。
- **仍待同學 Windows 實機驗證**：PowerShell 5.1 安裝、certreq key attachment、原生無帳密連線、DNS、內網、全流量和重連。PowerShell parser 與 package validator 通過，不能代替 Windows 實機。此部署未重新驗證兩台真實 Apple / Android 使用相同 client ID 的歷史互踢問題。

## 安裝包結構與使用者偏好（2026-10-06）

使用者要求直接在共用 VPN 根目錄放一個目前版本，其他放 archive。`<vpn-root>` 指 NFS 共用 home 下的 `dvlab-vpn`，現在只有：

```text
<vpn-root>/
├── DVLab-IKEv2-VPN.zip
├── DVLab-IKEv2-VPN.zip.sha256
├── DVLab-L2TP-VPN.zip
├── DVLab-L2TP-VPN.zip.sha256
└── archive/
    ├── latest/
    ├── releases/
    └── staging/
```

- 每個協定只提供一個根目錄下載包；不再讓成員選 latest、日期 release 或 staging。歷史目錄完整移入 archive，沒有刪除。
- 根目錄 IKEv2 包含 Windows 新憑證腳本、驗證腳本、兩張公開憑證、README、原 Apple 描述檔、原 Windows remover，共七個檔案。Apple profile 和 remover 與舊包逐位元組相同。
- IKEv2 ZIP SHA256：`1bff4aec5903b4d3f6d30e0c3c82fe1c4150d46f7ea08a795c8a2dc735aca3e8`。
- L2TP 根目錄保留原目前版本（原 2026-10-06-r2），SHA256：`9b214279f4f7c41f0dbdad6eada9cd3d416e49945e888f727fdb840422b68481`；此任務未改動 L2TP 內容。
- 根目錄的 checksum 使用新檔名，遠端 `sha256sum -c` 均通過；新版結構 validator 與三支 PowerShell parser 通過。根目錄发布是使用者要求的整理，不代表 Windows 實機已過。
- 公開 HackMD、Restricted Google Docs 的兩份維運文件及 zeus 私有 Markdown mirrors 已更新下載路徑；HackMD 完整 export 比對、Docs readback 和 Drive text export 都驗證過。機密不放公開指南；Google 與 local mirror 要各自驗證。
- `<vpn-root>` 不是 Git repository；安裝包是共享 operational artifacts，不要憑空替它建立 Git PR。更新時驗證內容、checksum、下載路徑與實際連線。

## Windows 腳本 PropertyNotFoundStrict 根因與處置

- **現象**：同學執行 `Install-DVLab-IKEv2.ps1` 時中斷，報錯 `Set-RasphonePreSharedKey : 在此物件上找不到屬性 'Count'。請確認該屬性存在。`（`PropertyNotFoundStrict`）。
- **根因**：
  1. 該同學使用的是 2026-07-17 的早期安裝包。早期腳本在 Windows 上嘗試寫入 PSK，函式 `Set-RasphonePreSharedKey` 包含 `$lines = Get-Content -LiteralPath $PbkPath -Encoding Unicode`。
  2. Windows 原生產生的 `rasphone.pbk` 常為 ANSI 或 UTF-8。強制以 Unicode 讀取會使換行無法識別，回傳單一 scalar 字串（或空白檔案為 null）。
  3. 腳本開頭設有 `Set-StrictMode -Version Latest`，存取 scalar 物件的 `.Count` 會直接觸發例外終止。
- **處置**：
  - 當時處置為改用 EAP 安裝包，不再呼叫 `Set-RasphonePreSharedKey`；後續舊腳本嘗試 PSK，但不代表原生連線成功；現行已改憑證，勿將此歷史處置當作現行安裝指引。
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
- 2026-09-28 公開 HackMD `DVLab VPN 安裝指南` 原仍教 Windows 執行舊 EAP 腳本並輸入共用帳密，與已停用的 EAP 不符。已改成 `.141` PSK 現況、告知 Windows 向網管取得設定，並同步本機 VPN 使用需知的五列常見問題；再次 export 與本機完整 Markdown 逐位元組相同。公開文件未加入 PSK 明文或私有網管設定。
- 2026-09-28 使用者指出 Windows 現行安裝方式就是腳本。已將公開 HackMD 指引與本機 VPN 使用需知改為「向網管取得現行 PSK 安裝腳本」，移除公開指南中引導 Windows 下載舊 EAP ZIP 的指令；HackMD export 與本機指南完整相同。正式 Restricted Google 文件的 Windows 段落也由 Docs API 增量加入這項更正，Docs 讀回與 Drive 純文字匯出確認新增文字且其他正文未變，owner-only 權限仍在。
- 2026-09-28 在 NFS home 找到 `<remote-home>/dvlab-vpn/releases/2026-07-26/DVLab-IKEv2-VPN.zip`：ZIP/sha256 一致，包內 Windows 腳本為 `.141` PSK、連線名稱 `DVLab IKEv2`，PSK 與 Apple 描述檔及私有防火牆文件一致；README 提供 PowerShell 安裝步驟。原 `vpn-release-tests/validate-release.py` 錯把字面 `0.0.0.0/0` 當作全流量條件；2026-07-26 腳本雖移除了 `Add-VpnConnectionRoute`，仍在 `Add-VpnConnection` 參數中明設 `SplitTunneling = $false`。驗證器已改檢查此值，2026-07-17、07-25、07-26 的 PSK 包均通過；將值改為 `$true` 的負例被拒絕。使用者確認此腳本能在現有 Windows 上執行。
- 2026-09-28 將版本化 2026-07-26 PSK ZIP 與 checksum 發布至 `<remote-home>/dvlab-vpn/latest/`，再讀回校驗、跑 release validator、比對原始檔完全一致。刪除 `releases/2026-07-26-eap` 的舊 EAP 包、較早的 `2026-07-17`／`2026-07-25` PSK 發布包及本機早期 PSK 腳本；`releases/` 僅保留 2026-07-26。公開 HackMD 指南改回從 `latest` 下載並讀回完整比對；Restricted Google 維運文件的 Windows 提醒與 NFS release 段落由 Docs API 修正，讀回正文、Drive 匯出及 owner-only 權限確認。

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
