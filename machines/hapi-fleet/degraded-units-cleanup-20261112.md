---
title: 實驗室四台 degraded 的三個常駐原因（logrotate、sssd 輪替、apport）與修法
scope: machines/hapi-fleet
tags: [systemd, logrotate, sssd, apport, gdm, ubuntu-26.04]
status: active
created: 2026-11-12
updated: 2026-11-12
---

2026-11-12 經使用者同意，在 mazu、cthulhu、valkyrie、athena 用 `su_<host>` + `sudo -n` 修正。四台修完後都是 `running`、0 個 failed unit。備份在各台的 `/var/backups/fleet-unit-fix-20261112/`。

- **logrotate 每天失敗**：Ubuntu 26.04 的 `cloud-init` 與 `cloud-init-base` 都裝了內容相同的 `/etc/logrotate.d/cloud-init*`，logrotate 回報 `duplicate log entry for /var/log/cloud-init*.log`，以 exit 1 結束。修法：刪掉 `cloud-init` 那份（它本來就依賴 `cloud-init-base`）；dpkg 不會把使用者刪掉的 conffile 裝回來。Athena 沒有裝 cloud-init，不受影響。
- **sssd 一直寫在已輪替的舊 log**：`/etc/logrotate.d/sssd-common`（屬於 sssd-common）的 postrotate 讀 `/var/run/sssd.pid`，但實際 pid 檔是 `/run/sssd/sssd.pid`。因為有 `|| true`，失敗時沒有任何錯誤，sssd 從沒收到 HUP，所以 mazu 和 cthulhu 一直寫到 `sssd.log.1`。修法：把路徑改成 `/run/sssd/sssd.pid`，再對 sssd 送一次 HUP，所有子程序就會回到目前的 log 檔。四台都已修改。之後 sssd-common 升級如果跳出 conffile 提示，要保留本機版本。
- **apport-autoreport 失敗（cthulhu）**：`/var/lib/apport/autoreport` 存在時，`/var/crash` 一有變動就會跑 `whoopsie-upload-all`。但 whoopsie 沒在運作，報告永遠上傳不了，服務逾時後以 exit 2 結束。修法：移除該旗標檔，關閉自動回報，與 athena、valkyrie 一致。沒有啟動 whoopsie，也沒有對外傳送 crash 報告。
- **gdm-greeter 的 user@6057x、session scope 失敗**：這些都是 [systemd-oomd 拖慢 user manager](earlyoom-nis-dynamicuser.md) 留下的殘骸。CreateSession 等了 2 分鐘逾時之後，gdm 改用 `gdm-greeter-2` 重試成功。oomd 移除後只要執行 `reset-failed` 就好。每台都有一個正常運作的 greeter。
