---
title: Inari Caddy LaunchDaemon 的日誌權限與安裝驗證
scope: machines/inari
machine: inari
status: active
updated: 2026-09-10
---

# Inari Caddy 啟動與公開 HTTPS

- 系統 LaunchDaemon 的 `UserName=_www` 遇 `spawn scheduled` 或 `EX_CONFIG` 且沒有 Caddy 日誌，先查 stdout/stderr 目標能否開啟。預先建檔 `_www:_www`、mode 640 再 `launchctl kickstart system/<service-label>`。此機已驗 _www 可監聽 80/443，讓 certificate state 目錄另為 _www 可寫、mode 700。官方 Caddy 2.11.4 使用 SHA-512 checksum，依該 release 核對 digest。
- HTTP localhost、LAN 或 NAT loopback 成功不能證明外部可信 HTTPS。分開驗外網 SYN、DNS、ACME、憑證和正式 URL，再發布 canonical。若同一外網探測對正常 IP 可在 WAN 匹配 SYN、舊站 IP 不可，查臺大 CERT 精確 IP 狀態和 WAN 記錄。曾重新安裝 OpenSSH 後仍在列管，須由登記網管依規定解除，不能猜 CVE 或認為升級後自動解除。歷史解除後，既有 Caddy 自行取得憑證，沒有重設 DNS、防火牆或服務。[臺大 CERT 精確 IP 查詢](https://cert.ntu.edu.tw/Module/Index/ip.php)
- 發布前確認 main／artifact 的實際 SHA。可採同一 GitHub Pages run 的 `github-pages` artifact 內 `artifact.tar`，避免兩站出不同提交；檢查成員路徑及 symlink，切換成品後保留舊 release 作 rollback。驗證首頁、主要語言頁、成員頁、robots/sitemap 和未知路徑 404。
- 網路狀態時好時壞時先查 `pmset -g log`。Maintenance Sleep／`Sleep Service Back to Sleep` 在 `sleep 0`、`standby 0` 下仍發生，Enet.Service DarkWake 後可恢復。駐站使用 `sudo pmset -a disablesleep 1`，查 `pmset -g` 和 IOPMrootDomain 各自顯示 1/Yes，避免改到螢幕 timer；撤回為 `sudo pmset -a disablesleep 0`。當時外網 TLS 已重驗、Caddy 程序未變；未做強制睡眠或重開驗證，也未查明更早進入 Maintenance Sleep 的觸發。 [Apple pmset source](https://github.com/apple-oss-distributions/PowerManagement/blob/main/pmset/pmset.m)

[各階段舊 IP、電源、部署與 HTTPS 收據](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/machines/inari/caddy-launchdaemon.md)
