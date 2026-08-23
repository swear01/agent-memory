---
title: Ubuntu release-upgrade recovery and authentication evidence
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-23
updated: 2026-08-23
tags:
  - ubuntu
  - release-upgrade
  - ssh
  - nis
  - nfs
  - recovery
---

# 已確認的證據界線

- 最近一次 fleet upgrade 記錄是多個獨立 worker 同時執行：四台主機做
  `22.04 → 24.04`，另一台做 `24.04 → 26.04`。因此後續同時出現的登入與
  control-plane 問題不能直接歸因於 Zeus 單一主機。
- 該記錄在 `24.04 → 26.04` 使用過 `do-release-upgrade -d`；`-d` 是 development
  release 選項，穩定版本維護窗口不得使用。
- 新的明確 TCP/22 探測顯示 Valkyrie、Athena、Cthulhu 可達；Mazu、Zeus
  探測逾時。逾時只能證明該次探測失敗，不能證明主機斷電或整機 freeze。
- 本機帳號互動式登入在 Valkyrie、Athena、Cthulhu 成功，且 `id -un` 與主機
  帳號一致；同時中央管理帳號的 SSH 公鑰流程在這三台到達 authentication
  boundary 後被拒絕。這把 host/network/SSH/local-auth 與中央 NIS/NFS/key path
  問題分開，但尚未單靠遠端證據證明具體 host-side 根因。
- Mazu 是 HAPI hub/tunnel 所在主機，Zeus 是 NIS master。兩者在同一事件窗口
  不可達時，會同時影響遠端控制與中央登入依賴；這是架構上的共同依賴，不是
  主機已關機的證明。

# 下次升級的最小流程

1. 每台主機先做 read-only preflight：OS/kernel、`dpkg --audit`、pending reboot、
   SSH enabled、network-online、`findmnt <remote-home>`、NIS/NFS readiness、GPU/DKMS、
   HAPI runner 與 workload。不要把 active SQLite/WAL 或事件輸出寫回 NFS home。
2. 對每台分別測試 TCP、SSH handshake、公鑰帳號與主機本機帳號；記錄邊界，不用
   單一 ping 或 HAPI 狀態代替全部證據。
3. Release upgrade 一次只跨一個受支援的 Ubuntu release；穩定窗口不用 `-d`。
   Package transaction 完成後仍必須 reboot，並在 reboot barrier 後重做全部檢查。
4. 若新 kernel panic、root filesystem mount 失敗或 boot loop，從 GRUB 選最後一個
   known-good kernel，先保留它，收集 failed-boot journal、initramfs、DKMS 與
   `dpkg` 證據，再修復；不要先 bulk remove kernel/driver。
5. 若本機帳號成功而中央帳號失敗，優先檢查 NIS binding/server、NFS `<remote-home>`、
   authorized-key 路徑與 SSH auth log；不要把它誤判成 GPU freeze 或斷電。

# 維運結論

升級的 package success、主機有電、TCP/22、SSH 認證、NIS/NFS、HAPI control
plane 是不同 gate。只有 post-reboot 的全部必要 gate 都通過，才把該台標成完成。
任何尚未在主機端 log 證明的根因都必須標為 hypothesis，並保留獨立 console/LAN/
BMC/PDU 的下一步驗證。

# 2026-08-23 Mazu 與 Zeus 現場恢復證據

- Mazu 從 GRUB recovery root 完成 `dpkg --configure -a`；之後 `dpkg --audit`
  無輸出、`apt-get check` exit `0`，並完成 `update-initramfs -u -k all`、
  `update-grub` 與 reboot。重開後 LAN `192.168.1.207` ping 及 TCP/22 可達，
  但 hub port `3006` 無回應，公開 HAPI 仍為 HTTP `530`。因此 OS package recovery
  有強證據，但 HAPI/NIS/NFS/GPU 等 post-reboot gate 尚未全部通過。
- Zeus 的 `140.112.171.138` ping、TCP/22、RPC/111 均可達，證明主機與 sshd 已
  上線；但 `rpcinfo -p 140.112.171.138` 只列出 RPC program `100000` (`rpcbind`)，
  未列出 NIS `ypserv` program `100004`。這是 NIS master 服務尚未向 rpcbind 註冊
  的直接證據，可解釋中央帳號驗證失敗；NFS `<remote-home>` 仍需獨立驗證，不得由此推定。
- 本地救援帳號及其密碼由私有 DVLab HackMD 管理。記憶只保留帳號路徑的存在與
  驗證結論，不保存或重述密碼。
- 使用 HackMD 管理的主機本地帳號重新做密碼 SSH 驗證後，Valkyrie、Athena、
  Cthulhu 三台皆成功登入，`id -un` 分別與主機名一致。現場版本為：Valkyrie
  Ubuntu `26.04` / kernel `7.0.0-30-generic`；Athena Ubuntu `24.04.4` / kernel
  `6.8.0-106-generic`；Cthulhu Ubuntu `24.04.4` / kernel `6.8.0-138-generic`。
  三台 `dpkg --audit` 均無輸出；但 `systemctl is-system-running` 分別為 Valkyrie
  `degraded`、Athena `starting`、Cthulhu `starting`，因此只證明本地登入與套件
  audit gate，尚未證明所有服務健康。

# 2026-08-23 Zeus kernel 7.0 最終恢復

- Zeus 掛載 Gaia 的 NFS `<remote-home>`，同時又從 Zeus 匯出相同 `<remote-home>`，使
  `nfs-server-generator` 產生 `home.mount` 與 `nfs-server.service` ordering cycle。
  已停用 Zeus 的 `<remote-home>` re-export 與 `nfs-server.service`；Zeus 保留 NIS master，
  不再充當這份 `<remote-home>` 的 NFS server。
- `<remote-home>` 的 `x-systemd.automount` 會被 `systemd-resolved`、`systemd-timesyncd` 的
  mount namespace 提前觸發，網路尚未 ready 時造成 90 秒 timeout 與
  `226/NAMESPACE`。移除 automount 後改由 `_netdev,auto,nofail` 正常掛載。
- 未接線的 `enp5s0` 使 `systemd-networkd-wait-online` 額外等待約 120 秒；Netplan
  只將該介面設為 `optional: true`，保留原 IP 設定。最終 kernel
  `7.0.0-30-generic` 開機為 58.283 秒，wait-online 4.540 秒，failed units 與
  `dpkg --audit` 均為空。
- 最終重開後已實測本機 Zeus 帳號、中央 `swear02` SSH、公鑰、Gaia NFS `<remote-home>`、
  `ypserv`、`ypbind`、`yppasswdd`、DNS、NTP 與 HAPI runner。HAPI runner PID 與
  `/var/tmp/hapi-zeus/runner.state.json` 一致，版本 `0.29.0.1`，hub heartbeat 正常。
- SSH 由 enabled `ssh.socket` 持久啟動；`ssh.service` 顯示 disabled 不能單獨判成
  SSH 未持久啟用。NIS 三項服務均 enabled。`yppasswdd` 會在密碼更新後執行
  `pwupdate` 重建 passwd/shadow maps；新增、刪除帳號或群組仍需在 master 執行
  `make -C /var/yp`，目前沒有 cron、timer 或 path watcher 自動完成這一步。
- 先前交接只規劃 SSSD 的 2–7 天離線憑證快取，沒有實際部署。Zeus 沒有 SSSD
  套件、服務或設定；Mazu、Valkyrie、Athena 只有 `libpam-sss`，沒有 SSSD daemon
  或 `/etc/sssd/sssd.conf`，NSS 仍使用 NIS。SSSD 可用 proxy 包裝 legacy NSS/PAM，
  但不能把未驗證的 proxy 設計當成 LDAP/AD/IPA 等原生 provider 的離線備援；
  不得把規劃文字當成已完成部署。
