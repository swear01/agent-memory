---
title: SwairM5 Kyocera ECOSYS M6635cidn 列印（IPP / 紙匣）
scope: machines/swairM5
status: active
created: 2026-09-29
updated: 2026-10-05
---

# Kyocera ECOSYS M6635cidn 列印注意事項

## 環境

- 機器：`SwairM5`（machineId `e02ab314-a679-4675-87b6-1d51914ed74f`）
- CUPS 佇列名：`Kyocera_ECOSYS_M6635cidn`（系統預設印表機）
- 驅動：AirPrint（`ECOSYS M6635cidn-AirPrint`）
- 位址：`192.168.1.100`（mDNS：`KMCC9F1B.local`）
- IPP：`ipp://192.168.1.100:631/ipp/print`（亦支援 `ipps://…:443/ipp/print`）

## Durable rule

1. **不要依賴 CUPS `InputSlot=`**：AirPrint PPD 的 `InputSlot tray-1/tray-2/by-pass-tray` 指令是空字串，`lp -o InputSlot=…` 送不到機器，常會落到 MP 匣錯誤。
2. **用紙匣時改走 IPP `Print-Job` + `media-col.media-source`**（`tray-1` / `tray-2` / `by-pass-tray` / `auto`），並指定 `media-size`（A4 直向 `x=21000 y=29700`，單位 1/100 mm）；margin 可採印表機已回報的預設值。
3. **彩色**：`print-color-mode=color`；黑白：`monochrome`。
4. **紙匣 2（Cassette 2）** 是常用補紙匣；MP tray 空時若工作指定到 MP，印表機會進入 `stopped` +「Check for appropriate paper in MP tray」，需在面板取消/清除，或改送正確紙匣的新 IPP 工作。
5. **docx → PDF**：本機有 Microsoft Word 時，用 AppleScript `save as … file format format PDF` 匯出，再列印；輸出可放 `~/Downloads/`。

## 驗證過的例子（2026-09-29）

- 工讀生簽到退表（docx→PDF）與 `DV_Lab_115_座位表.pdf`（彩色）皆以 IPP `media-source=tray-2` 成功完成。
- 座位表 PDF Drive id：`1ByCnoHK8mH89L863D_qsau2FAf3P4uzV`；本機副本曾放 `~/Downloads/DV_Lab_115_座位表.pdf`。

## 從實驗室 VPN 列印（2026-10-05 實測）

- 使用者指定的實驗室 VPN 是原生 IPSec `DVLab IKEv2`；不要代換成 `SwearOVPN`（OpenVPN）或 FortiClient 的 ADFP VPN。
- 本次原生 IKEv2 profile 在系統設定可見，但 `scutil --nc list` 沒有列出，`scutil --nc status 'DVLab IKEv2'` 回報 `No service`。這不代表 profile 不存在。2026-10-05 已設定原生捷徑，可用 `shortcuts run "DVLab 連線"`／`shortcuts run "DVLab 斷線"` 控制；見 [DVLab IKEv2 捷徑](dvlab-ikev2-shortcuts.md)。亦可由「系統設定 → VPN」操作既有設定。
- 可用 `open 'x-apple.systempreferences:com.apple.NetworkExtensionSettingsUI.NESettingsUIExtension'` 開啟該頁。連線後讀回「已連線」，並用 `route -n get 192.168.1.100` 確認實驗室位址走 `ipsec0`；不要只依 VPN 開關判定印表機可達。
- VPN 上 mDNS 查詢沒有取得印表機位址；直接查既有 IPP 位址，先以 `Get-Printer-Attributes` 核對型號與 UUID 是否匹配本機 CUPS 印表機，再送印，不必掃描整個網段。
- 經 `Validate-Job` 接受後，直接 `Print-Job` 指定 `media-col.media-source=tray-2`、A4、`copies=1`、`print-color-mode=monochrome`、`sides=one-sided`、`print-scaling=fit`，本次兩份既有 PDF 成功列印。
- 完成必須由印表機的 `Get-Job-Attributes` 回報 `job-state=completed`、`job-state-reasons=job-completed-successfully`，並核對 `job-impressions-completed` 等於 PDF 頁數；本機佇列接收或 HTTP 成功不足以證明印完。此次第二冊漢字詞練字紙第 13–18 課完成 30 頁，片假名練字紙完成 11 頁，最後印表機 `idle`、佇列 0。
- VPN profile、印表機位址、紙匣紙量與 PDF 版本都需下次重查；此完成紀錄不授權再次列印。
