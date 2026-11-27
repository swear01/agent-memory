---
title: Swear01_PC Windows TCP UDP 臨時連接埠耗盡調查
scope: machine
machine: swear01-pc
status: active
updated: 2026-10-24
---

# Windows TCP／UDP 連接埠分配失敗與音訊句柄累積

## 先保住事件證據

- Tcpip 4231／4266 證明 TCP／UDP ephemeral port allocation failure，System XML 的 Execution PID 4 是記錄者。透過當下 Get-NetTCPConnection（含 Bound、CloseWait、TimeWait）、Get-NetUDPEndpoint、PID＋StartTime、service、nonpaged pool、commit 和 AFD 交叉驗。快照在事件後數秒且各項非同時，低 endpoint 數不能排除瞬間尖峰或 allocator 狀態。AFD handles、總句柄、已綁定 port 數不可互換。
- 核對 IPv4/IPv6 TCP/UDP 動態範圍和 excluded ports。歷史觀察皆 49152–65535、只 60 個預留，未發現大幅縮小。TCP TIME_WAIT 不能直接解釋 UDP。完整 BootType 0 後和一般運行中均有失敗；Fast Startup 的 BootType 1 不等同重建核心。用 Kernel-Boot 27 / Power 42、107 / Power-Troubleshooter 1 分類，不能一律歸因於休眠。
- Sysinternals Handle 用 `handle.exe -accepteula -nobanner -a -v Afd`，trim CSV 欄名和文字，只計 `Type=File`、Name 前綴 `\Device\Afd`。比較音訊用相同 PID/StartTime 與 trim 後 Type/Handle/Name；枚舉列數等於總句柄仍可全部 Unknown，須另外驗分類。一般與提升權限的分類差不能當成新增／釋放。Store 更新後用目前 AppxPackage InstallLocation 解出 `Tools\handle.exe`，不要釘版本目錄。
- `Port Resource Monitor` 每五分鐘從 Interactive/Highest 取樣，`Port Exhaustion Capture` 訂閱事件。用 `wscript.exe //B //NoLogo` 的 hidden launcher，避免 PowerShell 窗口打斷使用者。登出、睡眠、開機早期可能無資料，主排程 `LastTaskResult=0` 不代表 audio 子收集成功；核對 errors.jsonl、CSV 檔大小和時間。手動擷取測試不能算自然復發證據。

## ETW 與故障指認

- 臨時 logman Winsock-AFD ETW 使用 256 MB circular buffer，事件觸發才凍結，沒有封包內容但含 PID/位址等中繼資料；跟原七天期限，不擅自延長。Session 不跨重開，Interactive 任務無法保證覆蓋開機第一分鐘。Freeze 在 trace 已停止時不可改 Captured 時間，核對實體 ETL 的時間與事件。記錄保存在原工作區 `outputs/port-monitor`，原私人 log 不進共享 repo。
- Bind 1029 的 `0xC0000209` 是 STATUS_TOO_MANY_ADDRESSES，`0xC0000043` 重複綁定對照不算。Payload Process 是 pointer，可能跨 socket 重用。用實際時間、HeaderPID、相對 endpoint 建立事件 1000 的 ProcessId 和行程起始時間比對。Brave network service 曾被證明是分配失敗的 caller；caller 不等於所有資源的消耗來源。觀察到的建立/關閉呼叫只是嘗試次數，不能當成功生命周期淨額。
- WARP 手動斷線仍可記 network-change reconnect，許多事件只重排 DNS/interface 列表。WARP 更新後 4231/4266 仍重現，和 Wi-Fi／resume 的時間相關不足以建立根因。事件快照未支持持續數萬 sockets 或整機耗盡；歷史存儲驅動分配失敗也無直接因果證據。未授權時保留現有 VPN 和監測。

## Discord／Audiodg 對照

- 同一 Discord renderer 在通話時重複累積 MMDevices Render/Properties Registry Key，集中 Steam Streaming Mic/Speakers 和 AM310。自然斷線後同一 renderer 的增長降到接近零，既有句柄未立即釋放。短期低增長之後仍曾再現；尚無 stack 證明特定音訊驅動、Experimental 或 port 耗盡因果。Audiodg 曾在既有授權關閉單一 endpoint 的全部 enhancements 後暫停增長，不能外推 Discord/系統根因。
- 使用者暫停永久固定 Standard，保留監測。當次手動 settings.json 後重啟，WebRTC `Creating audio engine: standard` 證明當次切換，但 renderer `Deferring` 及後續切回需要重新讀 log。官方 web bundle 的 automatic branch 屬機制線索，未核桌面同版，不能宣稱某個開關是持久解。需要對照就自然結束通話但保留 Discord 行程，比較至少 30 分鐘；不要中斷通話或同時改多個裝置。只通知新的可處理異常或有效對照。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/swear01-pc/windows-port-exhaustion-investigation.md)
