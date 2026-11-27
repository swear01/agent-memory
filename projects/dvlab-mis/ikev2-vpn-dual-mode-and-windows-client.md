---
title: DVLab IKEv2 VPN 現況與 Windows 用戶端排查
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-10-06
tags: [vpn, ikev2, windows, powershell, certificate, eap, psk, archive, dvlab-mis]
---

## 安裝包與認證

- **Windows 安裝腳本完成卻連不上。** 2026-10-06 的方案是 Windows 原生 IKEv2 `MachineCertificate`，Apple／Android 保持原 PSK，兩者共用原 VPN 入口。舊 Windows PSK 與退役 EAP 文件只用於查歷史。共用 `<vpn-root>` 的根目錄每協定只有一份目前 ZIP 及 SHA-256，歷史在 `archive/`。依 `sha256sum -c`、內容 validator 及 PowerShell parser 驗證發布。正式 Restricted Google Docs、公開 HackMD 各自讀回，公開指南不寫密鑰。
- **同 IP 的憑證協商卻落到 PSK 規則。** 在非 H 系列 Zyxel USG FLEX 200 V5.42，Phase 1 和 Phase 2 都要和既有 PSK proposal 區別；只改 DH 曾選錯規則。當時 PSK 為 AES256/SHA256，Windows 為 AES128/SHA256、DH14、no PFS、獨立 pool。若已連線只有內網，核對新連線也在 `IPSec_VPN` 與 `VPN_To_WAN_SNAT`。入口和雙 WAN 出口是不同驗收點，`write` 成功與 startup config 讀回分開。
- **Linux 或 parser 通過，就開放所有人使用。** enrollment ZIP 內含共用簽發密鑰，只發既有成員。Windows `certreq` 在 LocalMachine 生成不可匯出的私鑰，經 HTTPS 領個別憑證。公開 client CA 與 Zyxel server cert 分開釘選。外部 Linux 已驗證雙認證同時連線、重連、內網、直接 DNS 查詢、外網；系統 DNS 自動設定與 Windows 實機首裝、key attachment 和無帳密重連仍須分開驗證。

## 辨識舊故障

- **`Set-RasphonePreSharedKey`／`PropertyNotFoundStrict`。** 從公告的目前憑證版重取安裝包。舊版錯把 rasphone.pbk 固定當 Unicode，又對 scalar 或 null 讀 `.Count`。維護類似腳本時辨識 BOM／編碼，使用 `@(Get-Content ...)` 保持陣列；`SplitTunneling = $false` 也能宣告全流量，不靠是否出現 `0.0.0.0/0` 字串判斷。
- **RasClient 20226／828，五分鐘附近斷線。** 啟用管理員後用 `Get-NetIPsecQuickModeSA` 讀新連線的實際 `IdleDurationSeconds`，分開看 profile、ActiveStore、PersistentStore 和事件時間。舊 EAP 實測把全局與 RRAS 值設為 3600，重連及重啟 RasMan／IKEEXT 後 SA 仍 300；Windows 11 Home 的 `netsh ras set ikev2connection idletimeout=60` 亦回參數錯誤。仍無重新開機與長閒置對照，不能把這些操作當成功修法。

[原始紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/dvlab-mis/ikev2-vpn-dual-mode-and-windows-client.md)
