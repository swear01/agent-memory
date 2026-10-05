---
title: SwairM5 DVLab IKEv2 VPN 原生捷徑與命令列控制
scope: machines/swairM5
status: active
created: 2026-10-05
updated: 2026-10-05
tags: [DVLab, VPN, IKEv2, Apple-Shortcuts, shortcuts, macOS]
---

# DVLab VPN：連線與斷線

搜尋關鍵字：DVLab VPN、DVLab IKEv2、Apple Shortcuts、Apple 捷徑、VPN 連線 斷線、shortcuts run、scutil No service、vpnutil。

## 已設定的兩個動作

SwairM5 的 Apple「捷徑」已有以下兩個捷徑，皆選取既有 VPN `DVLab IKEv2`：

```sh
shortcuts run "DVLab 連線"
shortcuts run "DVLab 斷線"
```

使用原生 Set VPN 動作（`is.workflow.actions.vpn.set`），操作分別為 Connect 與 Disconnect；沒有 shell 動作或第三方工具依賴。macOS 命令列執行可能自動追加原生輸出動作，不能只依動作數量判斷內容。

2026-10-05 已移除 Homebrew `timac/vpnstatus/vpnutil`，移除後兩個捷徑仍通過實際連線／斷線驗證。不必重新安裝 vpnutil、另建 VPN profile 或寫 UI automation。

## 下次 agent 的使用與驗證

1. 先用 QMD `memory` collection 搜尋 `DVLab VPN` 或 `DVLab IKEv2`，讀取此文件；再以 `shortcuts list` 確認兩個名稱仍存在。
2. 使用者要求連線或斷線時，執行對應指令。捷徑完成不代表 VPN 已完成握手：輪詢 `scutil --nwi`，等待對應 VPN 的 `ipsec` 介面及 VPN server 資訊出現／消失；多條 VPN 並存時須核對系統設定中的目標 VPN 狀態。
3. 需要實驗室資源時，以該資源的路由及實際可達性驗證；不要僅憑 exit code 或 VPN 開關宣稱可用。測試後恢復使用者原本的連線狀態。
4. 捷徑需要登入中的 macOS 使用者工作階段；若畫面鎖定、互動授權或登入提示阻擋，讓使用者處理，不讀取或儲存憑證。

本次在登入工作階段以移除 vpnutil 後的兩次指令測試，確認 VPN tunnel 出現與消失，最終恢復未連線；不推論無人登入時也可操作。

## 為什麼 scutil 看不到

本機既有 profile 儲存在 NetworkExtension 設定，protocol 為 `NEVPNProtocolIKEv2`；`scutil --nc list` 走 SystemConfiguration 的 current network set，並非所有 NetworkExtension 設定的總表。本次名稱與 UUID 查詢都回報 `No service`，這不代表 VPN 不存在。

診斷時只讀設定的名稱、protocol 等必要欄位，不匯出原始 VPN plist 或鑰匙圈資料。

## 相關紀錄與來源

- [DVLab VPN 列印驗證](kyocera-ecosys-m6635cidn-ipp-print.md)：保留印表機 route／job completion 的獨立驗證。
- [Apple：從命令列執行捷徑](https://support.apple.com/guide/shortcuts-mac/run-shortcuts-from-the-command-line-apd455c82f02/mac)
- [Apple configd：SCNetworkConnectionCopyAvailableServices](https://github.com/apple-oss-distributions/configd/blob/main/SystemConfiguration.fproj/SCNetworkConnection.c)
