---
title: DVLab353 Zyxel DNS 改用臺大校級上游
scope: projects/dvlab-mis
project: dvlab-mis
status: active
created: 2026-09-15
updated: 2026-09-15
tags: [dns, zyxel, dvlab353, ntu, network]
---

# Zyxel DNS 上游（2026-09-15）

## 已完成且驗證的設定

使用者授權將防火牆 DNS 改為臺大校級 DNS。Domain Zone Forwarder 的兩筆 `*` 規則為：

1. `140.112.2.2`，Query via 保留 `wan1`。
2. `140.112.254.4`，Query via 保留 `wan1`。

LAN1／LAN2 DHCP 的 DNS 配置未變：第一個為 `8.8.8.8`，第二個為 `ZyWALL`；LAN1 裝置因此仍取得 `192.168.1.1` 作為另一個 DNS 選項。沒有修改 Wi-Fi、重啟或更新韌體。

經現行管理 API 及匯出的 `startup-config.conf` 雙重確認兩筆設定已儲存。透過 `192.168.1.1` 查 12 個網站的 A／AAAA，共 24 次全部成功；中位數 10.5 ms、最大 880 ms。這是當時的解析驗證，不能當作長期效能保證。

**尚未取得受影響手機的使用回報；不能記成所有手機開網頁慢都已修復。**

## 診斷證據與限制

- 原上游為 `140.112.171.254` 與 `1.1.1.1`，兩筆均為 `*`、經 `wan1`。7/17、8/23、9/9 備份皆相同；沒有建立者、首次設定時間或設定理由的證據。
- 現行 WAN1 設定確認 `140.112.171.254` 是出口閘道；被填入 DNS 欄位不代表它確實提供 DNS。Mac 對其 UDP／TCP DNS 查詢皆逾時，有線 Mazu 查詢也逾時。不能據此斷言防火牆自身來源的查詢必然逾時，也不能排除它過去提供過 DNS。
- 修改前，防火牆對 Netflix 的一次 AAAA 查詢耗時 2628 ms 後 SERVFAIL；同批 Google DNS 全成功。之後三輪 A／AAAA 重試正常，所以是觀察到的間歇異常，並非持續全面故障。
- 臺大官方校級 DNS 是上述新位址；現場直接查兩台均成功。來源：<https://isms.ntu.edu.tw/DNSlist.html>。
- 共用 DNS 是 Zyxel 的 relay／forwarder。當時 database 只列出全網域轉送，沒有內部名稱紀錄或專用網域轉送；未發現必須用它才能解析私有名稱的證據。不要把網管防火牆物件名稱誤當 DNS 紀錄。

## 與 Wi-Fi 事件的區別

9/11 的 R9000 無線瓶頸與 9/15 的手機開頁慢是不同事件，不能沿用前一次根因。現行 `DVLab353` 在 ASUS；NETGEAR 三個無線電仍關閉，僅供有線設備使用。參見 [R9000 Wi-Fi 與 ASUS 切換](netgear-r9000-wifi-slow.md)。

ASUS 日誌另有 DFS/CAC 暫停發送紀錄，但其 NTP 未同步，且部分紀錄位於無線初始化附近，無法與抱怨時間建立因果；本次沒有因此修改 DFS 或頻寬。

## 維運與文件

- 正式文件為共用網管 Google Drive 的 `353 network setup manual(private)`；已增量加入 Firewall configuration 下的 DNS 小節，保留原內容、Restricted 權限與文件身分。不要另建副本或更新已退役的私有 HackMD。
- 讀取用 `show ip dns server database`；兩筆修改沿用管理頁的 `ip dns server zone-forwarder <index> * user-defined <DNS> interface wan1`。以已登入的管理 API 寫入後，必須再讀回並核對啟動設定，不能只相信 HTTP 200。
- 若未來要還原，必須基於新診斷與授權；舊閘道 DNS 只是歷史備份，不是建議配置。
