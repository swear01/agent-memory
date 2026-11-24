---
title: 大量 SSH 輪詢會讓台大校園防火牆封鎖來源 IP
scope: tools/ssh
status: active
created: 2026-11-24
updated: 2026-11-24
tags: [SSH, NTU, firewall, IDS, polling, ControlMaster, dvlab, gups, 封鎖, 輪詢]
---

# 大量 SSH 輪詢會讓台大封鎖來源 IP

搜尋關鍵字：台大連不到、140.112 timeout、SSH 被擋、校園防火牆、brute force、輪詢太頻繁、ControlMaster。

## 發生了什麼（2026-11-24）

- 背景開了約 40 條等待鏈（CVSD 期末 APR → post-sim），每條每 30 秒開一次新的 SSH 連線去查 gups / dvlab（zeus、valkyrie、athena、mazu、cthulhu），同一 IP 每分鐘約 80 次登入。
- 之後家裡網路（Mac）連整個 140.112 都逾時：SSH、www.ntu.edu.tw、cool.ntu.edu.tw 都不通；同一時間清大、成大、NYCU 正常，外部檢測節點（check-host.net）連台大回 200。
- 判斷：校園 IDS 把來源 IP 當 SSH 掃描封鎖，範圍是台大全部服務。DVLab IKEv2 VPN 也連不上（伺服器同在被擋的網段）。

## 規則

- 一個批次只用一個輪詢迴圈，不要每個 run 一條。
- 每台主機重用一條連線：`ssh -o ControlMaster=auto -o ControlPath=~/.ssh/cm-%r@%h:%p -o ControlPersist=30m`。
- 輪詢間隔至少 2–3 分鐘；更好的是讓伺服器端寫完成標記，由單一迴圈檢查。
- 開批次前估算所有背景工作（含其他 session）加起來每分鐘新開的 SSH 次數，控制在約 5 次以內。
- 被封鎖時，使用者同意（2026-11-24）暫時用 Oracle VM 當跳板：只在 session 用的 `ssh -F` 設定裡讓 gups 走 `ProxyJump oracle`、dvlab 走 `ProxyJump gups`，不改 ~/.ssh/config，並保持 ControlMaster。

## 別跟「主機本身卡死」搞混（2026-11-24 晚）

- 封鎖的特徵：台大網站也連不上，TCP 直接逾時。
- gups（ieda1，140.112.186.18）卡死的特徵：ping 通、22/80/443/389 TCP 都能建立，但 SSH 停在 banner exchange、HTTPS 回 000、LDAP 無回應；從 Mac、Oracle、zeus、ieda2 內網（192.168.100.254）看都一樣。代表核心還活著、使用者空間卡住，要現場或主控台處理，換跳板沒用。
- ieda2（140.112.186.23）當時正常，可作為校內檢查點：`ssh root@140.112.186.23`。
- 依賴 `ProxyJump gups` 的 dvlab 連線會一起掛；dvlab 主機可以從校外直連時就改用直連。

### gups 卡死的原因調查

見 [[recurring-hang]]：先前「Innovus 用光記憶體」的推測已被推翻，前兩次卡死時機器是閒置的。
