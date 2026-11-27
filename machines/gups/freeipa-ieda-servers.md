---
title: "ieda1/ieda2 lab servers: FreeIPA, network, hardening"
scope: machines/gups
machine: gups
status: active
updated: 2026-11-28
---

# GUPS ieda1／ieda2：FreeIPA、NFS 和管理登入

- ieda1 是 FreeIPA master、ieda2 是 client，realm `GUPS.NTU.EDU.TW`。這套 FreeIPA 未整合 DNS，私網身份用 LAN hosts entries；公網 DNS 與本機 hosts 分著看。IPA ports 只在 firewalld internal LAN zone，公網維持已核准 SSH 路徑。xrdp/cockpit 的歷史啟停不代表現在開放。登入卡住先看 fail2ban、SSH 和 identity resolver，不能光看 gateway ping。
- ieda1 的 `/home` 透過 NFSv4 供 ieda2 LAN 使用，對應 export 限 ieda2、`root_squash`，NFS 防火牆只開 internal。ieda2 由 `/etc/auto.master.d/home.autofs`／`/etc/auto.home` 掛各帳號，SELinux `use_nfs_home_dirs` 開啟。區分 live NFS home 與舊本機合併備份，別當成兩份現役資料。NFS root_squash 下，root 不等於可讀 PDK；以授權帳號、group 與 actual file access 核對。
- 舊 ieda1 本機學生帳號未全部遷 IPA，不能因可登入 ieda1 就預期 ieda2 也可。IPA group `pdk_approved` 會小寫，GID 1024 是對應本機 `PDK_approved`；建立帳號與授 PDK 或 sudo 要分開驗。NDA 保護以 live 身分與群組為準，不擴大共享。LAN 隔離時 root/local account、系統磁碟和 SSSD lookup 可以仍正常，NFS 帳號的 home/access 仍因掛載重試延遲。
- `ipa-server-install` 曾在 umask 077 失敗，既有安裝要求 0022；不要把 admin password 放在 `ipa-client-install` argv。帳號建立只走受保護的管理入口和核准流程。寄送前可用原有 script 的 `--preview` 驗不會寄信，真實寄送需按使用者授權，憑證用保管庫與私有交接資料。
- ieda1 和 ieda2 的 TSRI license 為遠端 `lshc/lstn/lstc`；登錄的 public IP 必須符合當期 TSRI 規則。曾要求 PTR 結尾 `edu.tw` 且 A 正解回同一 IP。獨立 resolver 交叉查 A/PTR；DNS 並非 GUPS 自管 zone，依計中官方申請入口辦理。家目錄和 DNS 資料都先查 live，不以當時的等待回覆當現況。
- IPA online data 備份從 ieda1 到 ieda2 的受控位置，時間與 `ipa-backup` scope 查 cron，並另驗還原。Journald 持久化不保證日誌一直可寫；若學生同步登入失敗但 `systemctl` 顯示 SSSD active，照 [記憶體壓力與 SSSD worker](sssd-login-memory-protect.md) 查實際 worker、domain 與 `id`。

[ieda2 掛載設定、LAN 中斷對照與舊 home 合併證據](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/gups/nfs-home-ieda2.md)
