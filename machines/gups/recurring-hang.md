---
title: gups (ieda1) recurring silent hangs — evidence and suspects
scope: machines/gups
machine: gups
status: active
created: 2026-11-24
updated: 2026-11-28
tags: [gups, ieda1, hang, 卡死, POST 76, ASMedia, ST2000DM008, BIOS, netconsole]
---

# gups（ieda1）反覆卡死

## 三次卡死的共同樣子

- 11/10 22:19、11/17 14:44、11/24 10:59：系統日誌突然中斷，前面沒有 OOM、hung task、MCE、ATA 或 XFS 錯誤。`last` 看不到正常關機紀錄（11/16 14:49 那次 ieda2 也在同一分鐘重開，可能是停電或兩台一起被重開）。
- 11/24 卡死期間：ping 有回應、22/80/443/389 TCP 連得上，但 SSH banner、HTTPS、LDAP 都沒回應；使用者程式還活著（`timeout 14000` 在 13:57 照常送出 TERM 給 Innovus）。
- 前兩次卡死時機器幾乎閒置（那一小時只寫了 1–3 個檔案），所以不是工作量或記憶體造成的。11/24 剛好在跑 6 個 Innovus，其中 V21b 10:57 在 malloc 裡崩潰，應該是巧合，或者是同一個問題的早期徵兆。
- 11/24 按 reset 熱重開後，主機板 debug LED 停在 76（AMI：PCH/chipset DXE 初始化），螢幕沒有任何畫面；完全斷電再開，才走到 A0 並開機。

## 和 ieda2（同主機板，沒有卡死）的差異

- BIOS：gups F11（2025/07），ieda2 F12g（2026/03）。
- 系統碟：gups 是 ST2000DM008（Barracuda 桌機碟，一般認為是 SMR），ieda2 是 ST2000NM012B（Exos）。兩台都接在 ASMedia 1062（45:00.0）。gups 的 SMART PASSED，錯誤紀錄為 0。
- 其他設定相同：核心 4.18.0-553.179.1、tuned throughput-performance、SATA LPM max_performance、PCIe ASPM disabled、vm.dirty_ratio 40。

## 下一步

- 2026-11-24 已設好 netconsole：ieda1 透過 `netconsole-service`（`/etc/sysconfig/netconsole` 的 SYSLOGADDR=192.168.100.1，SYSLOGPORT=514，原檔備份為 `.orig`），並用 `/etc/sysctl.d/90-netconsole-printk.conf` 設定 `kernel.printk = 7 4 1 7`，連 INFO 等級也送出。ieda2 由 `/etc/rsyslog.d/10-netconsole-ieda1.conf` 只在 192.168.100.1:514/udp 接收，寫進 `/var/log/netconsole-ieda1.log`（logrotate 每週一次、保留 12 份），firewalld internal 區開放 514/udp。用 `/dev/kmsg` 測試，端到端成功；重開機後是否仍生效尚未實測。下次卡死時先看這個檔案。ieda2 若在 ieda1 開機時沒開，netconsole 會因為查不到 MAC 而啟動失敗，要 `systemctl restart netconsole`。
- 把 gups 的 BIOS 更新到 F12g，和 ieda2 一致；要到現場用 Q-Flash 更新。
- 長期把系統碟換成 NVMe。IPA、/home NFS 和 EDA 暫存目前都放在同一顆 HDD 上。
- 卡住時要完全斷電再開；只按 reset 可能會停在 POST 76。
- 2026-11-26 另有一次日誌停更：那是記憶體耗盡後 `sssd` worker 與 `systemd-journald` 掛掉（journald 撞上 StartLimit），不是這種沒有 OOM 紀錄的沉默卡死。見 [[sssd-login-memory-protect]]。

相關：[[campus-firewall-ssh-rate-block]]、[[freeipa-ieda-servers]]、[[sssd-login-memory-protect]]
