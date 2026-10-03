---
title: Swear01_PC Windows TCP UDP 臨時連接埠耗盡調查
scope: machine
machine: swear01-pc
status: active
updated: 2026-10-04
---

# 已驗證線索；根因尚未確認

- 2026-08-01 至 09-26 的 System 日誌有 Tcpip 4231 共 16 筆、4266 共 19 筆。事件 XML 的 Execution ProcessID=4 是記錄事件的系統行程，不能當成洩漏來源 PID。
- IPv4/IPv6 TCP/UDP 動態範圍均為 49152–65535；IPv4 只有 50000–50059 的 60 個動態埠被預留，沒有發現大量預留或縮小範圍。
- 09-04 23:15:17 完整啟動（Kernel-Boot 27=0x0）後，23:29:33 即出現 UDP 耗盡；當晚另一次完整啟動後約 17 分鐘出現 TCP 耗盡。不能將問題歸結成「只有連續開機十天才發生」。
- 啟用 Fast Startup；09-17 至 09-24 有八次 Kernel-Boot 27=0x1。關機後開機並不代表完整重建核心；排查驅動累積資源時應分辨完整啟動和快速啟動。
- 09-28 重啟後三次短時取樣，TCP 動態本機埠 62–65、UDP 40–41。完整 AFD 句柄快照約 279；WARP 1、ZeroTier 16、Radmin 7、RustDesk 4。這是正常時的基準，不能排除故障期間的洩漏。

## WARP 的具體異常

調查前版本 2025.10.186.0；daemon 運行但 tunnel 為 Manual Disconnection。歷史日誌仍反覆進入 `Reconnecting on network change`，不能把這行直接解讀成成功建立了新 tunnel 或新 socket。

對保存的四個輪替日誌比較 old_info/new_info 共 1,210 組，有 1,161 組顯示的網卡資料、DNS 集合和其他欄位相同，只改變網卡/DNS 清單順序。例如 09-26 10:35:28 發生此類處理，22 秒後記錄 UDP 耗盡。這是時間相關線索，尚不是因果證明。

09-27 01 時段有 145 次重連處理訊息；04:18、04:36 有 socket error 10055，04:41–04:42 daemon watchdog 終止。需優先追蹤 WARP 網路變更處理及其與虛擬網卡的互動，同時保留「它只是系統資源不足受害者」的可能。

Cloudflare 官方 2026-08-19 的 Windows GA 2026.7.1343.0 版本說明包含修復 GUI 在 IPC client 建立失敗時造成的 process leak／system resource exhaustion。沒有證據確認本機就是該 bug；不可宣稱更新已證明可修復本機 TCP/UDP 耗盡。

## 診斷方法與界限

- 組合 Get-NetTCPConnection（包括 Bound/CloseWait/TimeWait）、Get-NetUDPEndpoint、行程 PID/啟動時間/句柄、服務對應、記憶體池指標；在重現時保存，才能指認來源。
- Sysinternals Handle 枚舉 socket 要使用 `handle.exe -accepteula -nobanner -a -v Afd`，再篩 `Type=File` 且 `Name` 以 `\Device\Afd` 開頭。只執行預設檔案搜尋曾回傳 No matching handles；`-a` 後能讀取 socket。Afd 搜尋也可能命中登錄鍵名稱，必須篩掉。
- TCP TIME_WAIT 的說明不能單獨解釋 UDP 耗盡。擴大動態埠範圍或縮短等待時間只適用於經確認的負載模式，不能當作已找到洩漏根因。
- 09-28 已依使用者要求將 WARP 更新為 2026.7.1376.0；GUI/CLI 版本一致、CloudflareWARP Running/Auto，保留 Manual Disconnection。安裝 MSI 紀錄與 winget 均成功；不能當作根因已證明解決。
- 已建立 Windows 排程工作 `Port Resource Monitor`：每 5 分鐘以目前使用者 Interactive/Highest 取樣，日誌位於 `<user-home>/Documents/Codex/2026-09-28/no-more-older-messages-conversation-log/outputs/port-monitor`，每日資料保留 30 天；實際兩次執行成功，LastTaskResult=0。登出/睡眠不取樣。Codex heartbeat 每小時檢查，只通知新的可處理異常。
- 使用者回報直接啟動 PowerShell 的排程每五分鐘跳出視窗。已改由 `wscript.exe //B //NoLogo` 執行 `launch-hidden.vbs`，使用 `WScript.Shell.Run(command, 0, True)` 隱藏子行程並傳回退出碼；更新後排程實際取樣成功、LastTaskResult=0。排程动作不要直接啟動帶主控台視窗的 PowerShell。

## 更新後仍重現的證據

- 09-28 更新 WARP 後，21:56:20 又記錄 Tcpip 4231、22:18:30 又記錄 4266。已核對事件訊息明確為 global TCP/UDP ephemeral port allocation failure；不能宣稱更新 WARP 已解決耗盡。
- 21:32 至 23:07 的 22 筆五分鐘取樣，TCP 不同動態本機 port 約 66–210、UDP 32–45。事件後 AFD 快照也未見數萬 socket。這種取樣只能證明取樣當下數量低，不能排除短暫尖峰或未呈現在 endpoint 表中的分配問題。
- 此期間新版 warp-svc、WARP GUI、ZeroTier、Radmin、RustDesk 未顯示持續增加的 endpoint/句柄。音訊 audiodg 同一 PID 的句柄從 1022 持續升至 2507，值得另外追蹤資源釋放；其 TCP/UDP 均為 0，沒有證據將它直接連結到 port 耗盡。
- PrismLauncher 的 javaw 約 8.3 GB 私有記憶體、AFD 句柄 101 且兩次快照相同。記憶體高不等於已證實洩漏；AFD 數量不等於已綁定 port 數量。
- 23:21 起已在既有五分鐘排程加入音訊句柄類型統計，完整明細保留 baseline 與 latest；實際捕獲 audiodg 2652 個句柄（與行程總數一致），其中 Key 2206、Event 137。Key 中 1809 個集中在 Render/FxProperties 設定；登錄裝置屬性對應 PHL 276E8V / NVIDIA High Definition Audio。下一次取樣 Key 增至 2227，值得追蹤未釋放的音訊設定資源，但不等於已證明 NVIDIA 驅動錯誤或 port 耗盡原因。已讀取 audiodg 模組快照，當時未見非 Microsoft 公司模組。
- 訂閱 Tcpip 4231/4266 的 `Port Exhaustion Capture` 已於 09-28 23:53 經使用者重新確認 Windows UAC 後完成登記。語法與 XPath 匹配歷史事件驗證通過；手動執行成功保存 TCP/UDP/行程/AFD 明細，LastTaskResult=0。此次 RecentExhaustionEvents=0，不能當作耗盡重現；仍需等自然事件驗證自動觸發。原有五分鐘取樣及新增 audiodg 統計也已實際運行成功。
- Handle 全類型 CSV 的 `Name ` 欄名及部分文字有尾端空白；讀取名稱時應 trim 欄名和文字，避免誤判無名稱。部分 AFD 搜尋模式 CSV 欄名則沒有尾端空白。
- 09-29 約 00:08 使用者手動勾選 PHL 276E8V 的 Disable all enhancements；已核實該裝置 FxProperties 的 PKEY_AudioEndpoint_Disable_SysFx=1。個別音效原本均未勾選，不能將此當作總開關已停用。對照試驗基準與結果記在本機 audio-intervention.json。試驗期間 heartbeat 改為每 30 分鐘，回報初步結果後已恢復每小時。
- 09-29 00:39 完成初步對照：同一 audiodg PID/StartTime 在 00:08:41 至 00:39:08 的七筆後續取樣中，Key 固定 2719，30.4 分鐘增量 0；相較調整前約每五分鐘增加 50，累積已在該觀察窗停止。此結果支持音訊句柄累積初步改善，不能證明 TCP/UDP 耗盡或整機死機已根治。結果已記錄並將 heartbeat 恢復每小時，繼續檢查是否復發。
- 09-29 01:29 的延長觀察：調整後約 80.4 分鐘、18 筆取樣，同一 audiodg PID/StartTime 的 Key 在 2719–2725 間小幅波動，最新 2720，淨增 1；改善在這段觀察窗仍維持。調整後未出現新的 Tcpip 4231/4266，非分頁池約 826–848 MB。這些結果不足以將音訊問題與先前 port 耗盡建立因果關係，也不能保證更長時間不復發。

## 10/3 事件捕捉與循環追蹤

- 10/1 00:25 UDP 4266（RecordId 89067）、10/2 13:19 TCP 4231（89305）真實復發。事件排程自動觸發已驗證成功，分別約 1.4 / 3.1 秒後取得快照：TCP 169 / 155、UDP 82 / 92、AFD 358 / 192，可用記憶體約 13 / 22 GB。取樣沒有顯示大規模 socket 累積；根因仍未知，不能證實網路耗盡造成全機記憶體枯竭。
- 10/2 WakeTime 13:18:39，TCP 耗盡在恢復後約 24 秒；同時有 Wi-Fi 初始化與 WARP 介面變更。WARP 保留日誌在兩次事件當秒未找到 bind/socket/10055 錯誤；這些時間關聯不足以指認 WARP 或 Wi-Fi 驅動。10/1 UDP 事件前後兩分鐘未見相同恢復事件，不能將所有復發歸因睡眠恢復。
- audiodg 已重啟；10/3 約 54 分鐘 Key 50–51 穩定。新行程初始 CSV Key19/Fx0、目前 Key51/Fx4；跨日增量不能證明持續洩漏，且不可直接比較舊行程試驗基準。
- 在本機監測目錄新增 trace-control.ps1，以 logman 的 PortExhaustionAFD ETW session 記錄 Microsoft-Windows-Winsock-AFD（keyword 0x8000000000000000、level5）。bincirc 256 MB、64 KB buffers（4–16）、每五秒 flush；沒有啟用封包擷取，仍含位址與行程中繼資料。由既有隱藏五分鐘排程維護，重啟後可重建；真實 Tcpip 事件時停止並保留 port-afd.etl，停止後不自動重啟。七天期限記在 trace-config.json；到期由下一次排程停用。
- 實際測試成功記錄 loopback bind 事件1030及對應 HeaderPID；重複綁定測試記錄事件1029、NTStatus 0xC0000043，analyze-trace.ps1 能解析且不誤標為 0xC0000209 耗盡。trace-probe 檔案是測試，不能算新耗盡。正式 trace-state=Running，排程 LastTaskResult=0。測試時 DateTime 與 DateTimeOffset 比較錯誤已修正，以 DateTimeOffset.Now 比較期限。
- 後續查 bind 失敗 0xC0000209（STATUS_TOO_MANY_ADDRESSES），結合 endpoint建立事件與快照行程啟動時間；payload Process 是指標，不能直接當 PID，HeaderPID 也需相關事件佐證。失敗呼叫者未必是消耗資源的來源。沒有匹配失敗不代表保留窗之外從未耗盡。
- 10/4 01:11:35 的新 TCP4231（RecordId89597）已成功觸發凍結循環 ETL。AFD事件1029在同一時間回傳 0xC0000209，HeaderPID與同一endpoint建立事件1000的payload ProcessId相符，均指向 Brave network.mojom.NetworkService；快照啟動時間確認是同一行程。這證實 Brave 網路服務是本次分配失敗的呼叫者，不能證明它消耗所有port；當下 TCP238、UDP90、AFD最高javaw102/Brave51、可用記憶體約11GB，audiodg Key51仍穩定。追蹤狀態Captured且Enabled=false，保留檔案未重啟。失敗前同一指標曾用於UDP，後來重新建立TCP；指標會重用，必須以時間與建立事件判讀，不能混算endpoint生命周期。低影響下一步是使用者方便時保存工作、重啟Brave做對照，不能直接停用VPN或宣稱Brave是耗盡根因。
- 10/4 使用者要求繼續追蹤後，已封存該次256MB ETL，01:42重新啟動下一輪循環檔，仍維持原七天期限。後續分析：保留窗00:55:33–01:11:36約16.05分鐘，Brave socket建立與關閉呼叫各1716（建立TCP703、UDP1013）；這些是進入事件的呼叫次數，不能當成功數或精確生命周期結餘。三小時同一行程取樣TCP27–58、UDP9–27、句柄548–769、PrivateMB32.2–38.1，未支持簡單持續累積未關閉socket的解釋。動態範圍重查仍16384、動態區間預留僅60，不是範圍被縮小。下一步比較新事件失敗呼叫者是否仍為Brave，暫未改其設定。
## 官方參考

- Microsoft Learn: TCP/IP port exhaustion troubleshooting
- Microsoft Learn: Delivering a great startup and shutdown experience
- Cloudflare One Client Changelog: Windows GA 2026.7.1343.0, 2026-08-19
