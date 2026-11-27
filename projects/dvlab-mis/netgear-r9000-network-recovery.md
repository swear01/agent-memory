---
title: DVLab353 R9000 管理面故障、Wi-Fi 瓶頸與 ASUS 上聯
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-11
---

- **重開後無改善，或看到 ASUS WAN 紅燈。** 先核對品牌、機身與管理入口；R9000 是有線交換，ASUS 用 LAN 上聯與 `DVLab353` SSID，WAN 燈不能單獨判斷。R9000 設定內可能保留同名 SSID，但 2.4／5／60 GHz radio 須維持關閉，否則撞名。不要 factory reset，會改 DHCP。地址、綁定和 Wi-Fi 現況以私人網路文件與現場讀回為準。
- **R9000 不回 ARP／管理頁，但下接伺服器仍通。** 分開量 L2/有線轉送、管理面和 Zyxel `ipmac-binding`。舊事件補入 LAN1 靜態 binding 後管理面仍不通，真正斷電重開 R9000 才恢復。故障不等於整機斷電／CPU 過熱；舊備份沒有該 binding，不能推定有人刪除。復發先保存當前 ARP、logs、診斷包及設定；重開後的 `panic_log.txt` 全 0xFF 代表無有效 panic，有限循環日誌和未啟用的 syslog 不能證明過去沒事。
- **有線正常，手機或 Wi-Fi 卻慢。** 同時量有線 WAN、無線 LAN/WAN、RSSI、實際協定、channel/width，而非只看管理頁設定。09-11 的 LAN/WAN 無線速度接近，實際只關聯 802.11n/40 MHz，即使管理頁顯示 1733 Mbps。當時切到 ASUS 802.11ax；09-15 手機慢另見 [校級 DNS 上游](firewall-dns-school-upstreams.md)，不能沿用同根因。
- **R9000 韌體 CGI 回錯或永遠等不到 finish.txt。** 有線端只留單一 admin 操作；`multi_login.html` 先依既有管理流程接手。multipart 順序先 `mtenFWUpload` 映像再 `upfile`，預檢看 `UPG_version.htm` 與 `new_version`，不是 finish.txt。確認同型號官方映像與備份後，授權 flash 才送 `upload_firmware`／`upgrade_yes_no=1`；變更後讀回三個 radio off 及有線正常。配置備份含機密，權限 0600、不進 Git；安全韌體更新不等於修了 channel width。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/dvlab-mis/netgear-r9000-network-recovery.md) · [Wi-Fi 切換與韌體原始驗證](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/dvlab-mis/netgear-r9000-wifi-slow.md)
