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

升到 `V1.0.6.46` **不會**修好 40 MHz；官方釋出說明只有 Security Fixes。

2026-09-11 19:08 已切換：ASUS 主 SSID 改名 `DVLab353`，R9000 2.4 / 5 / 60 GHz **無線關掉**（有線仍在）。這台 Mac 隨即連到 ASUS：802.11ax、ch104、160 MHz、PHY 1921 Mbps。

## 設備現況

- NETGEAR R9000：`192.168.1.11`，韌體 **`V1.0.6.46WW`**。有線 AP 仍開（LAN `192.168.1.11`、DHCP 仍關）。**無線已關**：`old_endis_wl_radio=0`、`old_endis_wla_radio=0`、`con_endis_wig_radio=0`。設定裡 SSID 名稱仍叫 `DVLab353`，只是沒在播；若有人把 Radio 再打開會跟 ASUS 撞名。
- ASUS TUF AX6000：`192.168.1.10`，2.4 / 5 GHz SSID 都是 **`DVLab353`**（2026-09-11 19:08 從 `DVLab353-2` 改名，Smart Connect 仍開）。這台 Mac 量到 802.11ax / ch104 / 160 MHz / Tx 1921 Mbps。
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

R9000 只當有線交換，Wi‑Fi 由 ASUS 用 SSID `DVLab353` 提供。不要把 R9000 Radio 再開回來（會跟 ASUS 同名互搶）。不要 factory reset。

## ASUS 改名 / R9000 關無線（已驗證）

- ASUS 網頁從 Mac 登入會 `error_status=4` 鎖幾分鐘；改從 **Mazu 有線** `login.cgi`（完整表單欄位）再 `applyapp.cgi`：`wl0_ssid`/`wl1_ssid`=`DVLab353`、`rc_service=restart_wireless`。回 `{modify:1, run_service:restart_wireless}` 後等約 25 秒。
- R9000：`POST apply.cgi?/WLG_adv.htm timestamp=…`，`submit_flag=wlan_adv`，`wl_enable_router`/`wla_enable_router`/`wig_enable_router`=`0`。不要送 `enable_ap` checkbox。成功後三個 `old_endis_*_radio` 都是 `0`。有線不受影響。
