---
title: DVLab353 變慢是 R9000 Wi-Fi 而非 WAN
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-11
tags: [network, netgear, r9000, wifi, zyxel, firmware]
---

# DVLab353 同學反映變慢：瓶頸在 R9000 無線

## 現場結論（2026-09-11）

同學感覺 router 慢，量測後瓶頸是 NETGEAR R9000 的 SSID `DVLab353`，不是 Zyxel WAN。Mazu 有線 Cloudflare 50 MB 約 464 Mbps、閘道 RTT 0.25 ms；同一時段 Mac 走 DVLab353 只有約 170 Mbps WAN 與 183 Mbps LAN。兩段幾乎相等，所以慢在 Wi-Fi hop。

該 Mac 訊號 −43 dBm，卻只關聯 802.11n、頻道 44、40 MHz、MCS 15、PHY 300 Mbps。R9000 管理頁 5 GHz 設為 Up to 1733 Mbps，實際 BSS 沒談到 80 MHz AC。

升到 `V1.0.6.46` **不會**修好 40 MHz；官方釋出說明只有 Security Fixes。速度問題仍要改頻道 / 關 Smart Connect / 請同學走 `DVLab353-2`。

## 設備現況

- NETGEAR R9000：`192.168.1.11`，韌體 **`V1.0.6.46WW`**（2026-09-11 18:24 由 Mazu 有線手動升級；先前 `V1.0.5.42WW`）。`currentsetting.htm` 的 `isBlankState=0`。WAN / `InternetConnectionStatus=Down` 在 AP 模式是預期狀態。
- 升級後設定仍在：LAN `192.168.1.11`、SSID `DVLab353`、DHCP Server 仍關（`LAN_lan.htm` 的 `if ('0' == '1') dhcp_server.checked`）。沒有 factory reset。
- 2.4 GHz 與 5 GHz SSID 都是 `DVLab353`，Smart Connect 開啟。
- ASUS TUF AX6000：`192.168.1.10`，SSID `DVLab353-2`。升級重開期間這台 Mac 改連到 802.11ax / ch104 / 160 MHz / Tx ~1921 Mbps（ASUS 特徵）。
- Zyxel CPU 約 6%、記憶體 39%。session 約 4059 筆，多數是 UDP/111，不是 Wi-Fi 變慢的原因。

## 干擾

Mac 掃描：5 GHz BSS 23、UNII-1（36–48）11、ch44 上 5 個。ASUS 在 ch40 160 MHz，與 R9000 ch44 重疊。

## 手動升級 R9000 的坑（已驗證）

- 必須從 **R9000 後方有線主機**（Mazu）上傳；Mac 走 `DVLab353` 時閃過會斷。
- 同時只能一個 admin。被踢到 `multi_login.html` 時 GET `change_user.html`（401 是預期），再重試。閃過期間不要從其他裝置打管理頁。
- 表單欄位順序：先 `mtenFWUpload`（`.img`），再 `upfile`。反過來 CGI 把檔名文字當映像，`upgrade_check.cgi` 立刻 `next=UPG_failure.htm`。
- 檢查成功：`next=UPG_version.htm` 且頁面出現 `var new_version="V1.0.6.46"`。不要等 `finish.txt==1`（成功時它常一直是空的；瀏覽器靠 1 秒 meta refresh）。
- 真正閃過：`POST /upgrade.cgi? timestamp={ts}`，`submit_flag=upload_firmware`、`upgrade_yes_no=1`。回 `UPG_process.htm` 後會重開。
- 禁止按 Reset / restore factory；會重開 DHCP、打亂 `192.168.1.0/24`。
- 官方檔：`R9000-V1.0.6.46.img`（41357441 bytes）。設定備份 `NETGEAR_R9000.cfg` 含密文，權限 600，不要 commit。

## 操作界線

同學日常最好走 `DVLab353-2`。改頻道、關 Smart Connect、拆開 2.4/5 GHz SSID 或關掉 R9000 無線都要另開維護。
