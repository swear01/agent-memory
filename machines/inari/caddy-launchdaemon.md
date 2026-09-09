---
title: Inari Caddy LaunchDaemon 的日誌權限與安裝驗證
scope: machines/inari
machine: inari
status: active
updated: 2026-09-09
---

# 已驗證的啟動條件

- macOS 26.6.2 的系統 LaunchDaemon 使用 `UserName=_www` 執行 Caddy 時，
  `StandardOutPath`／`StandardErrorPath` 指向的日誌若位於 `_www` 無法建立
  檔案的目錄，可能反覆停在 `spawn scheduled`、`last exit code = 78: EX_CONFIG`，
  而完全沒有 Caddy 日誌。先建立日誌檔並設為 `_www:_www`、mode 640，再
  `launchctl kickstart system/<service-label>`，本次成功啟動。
- Caddy 2.11.4 官方 release 的 `checksums.txt` 使用 SHA-512；不可假設為
  SHA-256。下載的 macOS arm64 archive 已用 SHA-512 成功核對。
- 此機實測 Caddy 可由 `_www` 帳號監聽 80／443；不必讓 Web server 以 root
  執行。憑證資料目錄須另設為 `_www` 可寫、mode 700。
- 常駐網站需要停用 AC 自動睡眠；本次原設定為 1 分鐘。`pmset -c sleep 0`
  不會停用螢幕睡眠。

# 驗證邊界

服務啟動、內網 HTTP 成功、或 NAT loopback 成功，都不能代替真正外網 HTTPS
驗證。憑證尚未簽發時必須分開回報部署成功與公開站未就緒，不能先發布指向
不可用主站的 canonical。網路是否正常需每次重新測量。

# 舊網域 IP 管制根因（2026-09-08 查證）

- 網站舊公網 IP `140.112.171.142` 在臺大 CERT 的「目前違規紀錄」及
  「OTHERS 受到 IP 管制中的主機」均有紀錄：2025-08-28 15:59，
  `Others-openSSH漏洞`。公開頁未列 CVE，不能自行推定漏洞編號或修復狀態。
- 官方來源：<https://cert.ntu.edu.tw/Module/Index/ip.php> 的精確 IP 查詢；
  當日名單位於 <https://cert.ntu.edu.tw/Module/Index/blocklist.php?type=others&page=4>。
  分頁及列管狀態會變動，後續恢復工作須重新查證。
- 同一外網節點探測時，防火牆 WAN1 能收到送往正常公網 IP 的 SYN，卻收不到
  舊站 IP 的 SYN；舊站出站封包已正確 NAT 後離開 WAN1，但無回包。不要再把
  內網 NAT loopback 成功當成校網已解除封鎖的證據。
- 實機 `sshd -V` 為 OpenSSH_10.3p1，macOS 26.6.2 (25G83)；重灌之後仍在
  列管名單。需由登記網管按官方流程確認原事件修復並申請解除，不能藉換 IP
  或代理來規避尚未解除的資安限制。

# 解除管制後恢復（2026-09-09 實測）

- 網管回覆重新啟用後，Caddy 不需改設定或重新啟動即取得可信憑證；
  inari 連 ACME directory 恢復 200，Moldova／Sweden 兩個海外節點對
  原網域 HTTPS 均回 200，解析仍為原 IP。恢復未改 DNS 或防火牆。
- 更新網站時需先同步 GitHub 最新 main，避免發布解除管制前的舊成品。
  靜態 release 採日期與提交編號命名；原 release 保留供回復。
  主站首頁、中英文頁、成員頁、robots 與 sitemap 和建置成品逐位元一致，
  不存在的路徑回 404。GitHub SEO 已透過 PR #84 合併發布，線上 canonical 指向原網域。
- 發布期間 main 可能有其他提交；同步主站時可下載該次 Pages workflow 的
  `github-pages` artifact，解開內含 `artifact.tar` 部署至 inari，避免兩站
  由不同提交建置。切換前先檢查 archive 不含路徑穿越或符號連結。
