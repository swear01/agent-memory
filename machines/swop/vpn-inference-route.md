---
title: Windows split-tunnel VPN 遺漏推論服務路由
scope: machines/swop
status: active
updated: 2026-09-21
---

VPN 顯示 Connected 不代表內網推論端點走 VPN。Pi 的 ninfer 請求曾連續 Request timed out；Find-NetRoute 顯示端點仍走 Wi-Fi default route，VPN profile 只有其他網段。

用 Find-NetRoute -RemoteIPAddress <inference-ip> 驗證實際出口；用 Get-VpnConnection 檢查 Routes。Add-VpnConnectionRoute -ConnectionName <vpn-name> -DestinationPrefix <inference-ip>/32 -RouteMetric 1 可加入只涵蓋服務主機的 profile route。重新連線後再次確認實際出口，不能只依 cmdlet 成功判定生效。本次重連後 Node.js chat completion 回 HTTP 200 並回答 OK。

swop 的 pi-safe.ps1 原本就加入 --use-bundled-ca 與 --use-system-ca，並從使用者 DPAPI 儲存載入 BIFROST_API_KEY。單獨執行裸 node fetch 會因 UNABLE_TO_VERIFY_LEAF_SIGNATURE 失敗；驗證應沿用 wrapper 的 CA 與憑證來源。不要把裸 Node 或 Windows curl 的憑證錯誤誤判為 Pi 設定故障，不需要關閉 TLS 驗證。
