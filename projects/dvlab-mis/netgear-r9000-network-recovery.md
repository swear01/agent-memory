---
title: DVLab353 NETGEAR R9000 recovery and diagnostic limits
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-09
---

# DVLab353 NETGEAR R9000 網路恢復

## 設備辨識

- NETGEAR R9000：SSID `DVLab353`，管理 IP `192.168.1.11`。
- ASUS TUF AX6000：SSID `DVLab353-2`，管理 IP `192.168.1.10`。
- Zyxel 防火牆：LAN `192.168.1.1`，管理 HTTPS 埠 `4433`。
- ASUS 最左側為 WAN 燈、最右側為電源燈。實驗室文件採 LAN 接上游、停用 router DHCP；WAN 紅燈不能單獨證明故障。不要把 ASUS 照片套用 NETGEAR 燈號說明。
- 這次使用者起初重開的是 ASUS；照片確認設備後，才真正重開 NETGEAR。下次先核對品牌、機身與 IP，再判斷重啟效果。

## 已驗證事件與修正

- 故障時 Mac、Zeus、Athena 無法取得 NETGEAR ARP 回應，管理入口不通；下接伺服器仍可通訊。擷取到 NETGEAR MAC 的底層封包，不代表管理系統正常，也不能直接斷言整機斷電或 CPU 當機。
- Zyxel 曾記錄 NETGEAR 的 `ipmac-binding` 丟包。2026-09-09 新增並持久儲存 LAN1 靜態綁定：`192.168.1.11` / `78:D2:94:58:6A:D8`，pool `netgear_dvlab353`，description `NETGEAR_DVLab353`。
- 回讀確認原九筆靜態設定保留，新增綁定進入有效表，日誌出現 `Insert hash item`。
- 2026-07-17 autobackup 與 2026-08-23 lastgood 都沒有 NETGEAR 固定綁定。新增後 startup-config 與 8/23 備份只差本次綁定及存檔時間：沒有近期刪除該綁定的證據。不得將「目前缺漏」敘述為「被人改掉」。
- 補綁定後仍無法登入；真正斷電重開 NETGEAR 後，ping、HTTP Basic 登入與 Logs 頁恢復。當時韌體為 `V1.0.5.42`。
- 根因未確認。不能把綁定缺漏、過熱、資源耗盡或韌體問題當作已證實原因；一次恢復也不等於長期穩定。

## 日誌與後續界線

- 重開後 Logs 僅見本次初始化、裝置連入及管理員登入。
- 已從診斷頁匯出 debug-log.zip；內含設定檔及 `panic_log.txt`。後者為 262144 bytes、全部 `0xFF`，沒有有效 panic 記錄。
- 當時開機自動 Debug Log Capture、Email 通知均未啟用，診斷儲存位置預設 System Memory。未取得重開前可判定原因的紀錄。
- Zyxel 當時可讀 1024 筆循環日誌，僅涵蓋約數分鐘；遠端 syslog 與 USB 日誌未啟用，不能期待完整變更歷史。
- 使用者決定暫緩 USB 持久日誌與 Zeus 監測；本次未啟用這些功能、未升級韌體、未恢復原廠設定。
- 下次復發先保存現有 log、診斷包與連線證據；不要先清除日誌或按 Reset。讀取診斷頁不等於已啟用持久收集。
- 重新處理時必須現場重驗連線、綁定及設備狀態。本筆是 2026-09-09 的歷史結果。
