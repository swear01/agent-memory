---
title: DVLab353 變慢是 R9000 Wi-Fi 而非 WAN
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-11
tags: [network, netgear, r9000, wifi, zyxel]
---

# DVLab353 同學反映變慢：瓶頸在 R9000 無線

## 現場結論（2026-09-11）

同學感覺 router 慢，量測後瓶頸是 NETGEAR R9000 的 SSID `DVLab353`，不是 Zyxel WAN。Mazu 有線 Cloudflare 50 MB 約 464 Mbps、閘道 RTT 0.25 ms；同一時段 Mac 走 DVLab353 只有約 170 Mbps WAN 與 183 Mbps LAN。兩段幾乎相等，所以慢在 Wi-Fi hop。

該 Mac 訊號 −43 dBm，卻只關聯 802.11n、頻道 44、40 MHz、MCS 15、PHY 300 Mbps。R9000 管理頁 5 GHz 設為 Up to 1733 Mbps，實際 BSS 沒談到 80 MHz AC。

## 設備現況

- NETGEAR R9000：`192.168.1.11`，韌體 `V1.0.5.42`，uptime 約 49.7 小時（對得上 2026-09-09 斷電重開），CPU/2G/5G 約 65°C。LAN 埠皆 1000M/Full。WAN / `InternetConnectionStatus=Down` 在 AP 模式是預期狀態。
- 2.4 GHz 與 5 GHz SSID 都是 `DVLab353`，Smart Connect 開啟。
- ASUS TUF AX6000：`192.168.1.10`，SSID `DVLab353-2`。2026-09-09 無線狀態為 HE 160 MHz，客戶端 PHY 1201–2401 Mbps。
- Zyxel CPU 約 6%、記憶體 39%。session 約 4059 筆，多數是 UDP/111，不是 Wi-Fi 變慢的原因。

## 干擾

Mac 掃描：5 GHz BSS 23、UNII-1（36–48）11、ch44 上 5 個。ASUS 在 ch40 160 MHz，與 R9000 ch44 重疊。

## 操作界線

先請使用者改連 `DVLab353-2`。改頻道、關 Smart Connect、拆開 2.4/5 GHz SSID、升級韌體或關掉 R9000 無線都要另開維護。官方較新韌體為 `V1.0.6.46`（2025-09）；現場當時仍是 `V1.0.5.42`。
