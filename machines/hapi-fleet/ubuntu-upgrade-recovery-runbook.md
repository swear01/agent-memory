---
title: Ubuntu release-upgrade recovery and authentication evidence
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-23
updated: 2026-11-19
tags:
  - ubuntu
  - release-upgrade
  - ssh
  - sssd
  - nis
  - nfs
  - recovery
---

# Ubuntu 升級復原與 NIS/SSSD 驗證

- 一次只升一個支援中的 Ubuntu release，穩定維護別用 `do-release-upgrade -d`。先查 audit、pending reboot、NIS/NFS、GPU/DKMS、workload、SSHD socket 和 HAPI。Zeus 是唯一 NIS master，Mazu 是 Hub/Tunnel，分批保留本機備用入口。
- 重開後分開驗本機帳號、TCP/22、SSH handshake、中央公鑰／密碼、NIS、NFS、GPU、HAPI。SSH socket enabled 而 service disabled 可正常。連線逾時不能當成斷電。若新 kernel 有 panic、mount root 失敗或 boot loop，先從 GRUB 退回 known-good kernel，保存 failed journal、initramfs、DKMS 和 dpkg 證據。Apt 交易修復不代表 post-reboot gate 通過。
- Zeus 曾把 Gaia 的共享 home 又從本機 re-export，讓 `home.mount` 和 `nfs-server.service` 循環，修法停重複 re-export。Home automount 曾被 DNS/NTP 的 mount namespace 過早觸發，改用 `_netdev,auto,nofail`；未接線介面按驗證結果設 optional。RPC 111 可達仍須查 ypserv `100004`。增刪 NIS 帳號／群組要在 master 重建 maps；密碼更新有另外的 pwupdate 路徑。救援憑證從私有 DVLab MIS 文件取用。
- Zeus 重開使用本機維護連線上的 `sudo /usr/local/sbin/zeus-safe-reboot`。先結束占用 `/apps` 和 `/home` 的工作，入口遇 busy／逾時／巢狀掛載會拒絕並還原掛載。直接 reboot 繞過檢查。腳本模擬、ShellCheck 和安裝驗證通過，當時未執行實機卸載或重開演練。

## 2026-11-19 Zeus 失聯與 NIS 查詢逾時

NIS 斷線診斷與七天認證快取適用範圍如下。

- Zeus 保持 NIS-only master。四台 GPU client 經 SSSD 代理 NIS，更新後 NSS 順序見下方 11/22。utux 有 SSSD identity gate，不能外推為同一 PAM 密碼修補。SSSD proxy/NIS 呼叫同步 libnsl RPC，底層可單次等 25 秒及重試；收到 UNAVAIL 才轉 offline。兩秒 offline retry 是恢復週期，不能保證首次 RPC 上限。NSS 先走 nis 時，本機帳號也可能卡在補充群組。
- `offline_credentials_expiration = 7`、`cached_auth_timeout = 604800`。後者使 ypbind 返回 PAM_AUTH_ERR 時仍能用先前驗過的 password；在線也可能接受舊快取。需立即撤銷時逐台執行 `sss_cache -u <user>` 失效帳戶快取，並實際驗證舊密碼拒絕。Auth/account primary block 需有 pam_sss，本機用 pam_unix fallback；SSSD 不接 password change。驗離線公鑰前以 SSSD 限定 lookup 暖 identity，再用 `sssctl user-show` 確認。
- 測離線保留本機管理連線和備份，對 Zeus LAN／公網 endpoint 做 DROP 並查計數，測未暖、已暖、過期帳號和新 SSH/sudo；解除後確認 `ypwhich`、即時線上資料。停 ypbind 只能快速 UNAVAIL，且反覆啟停可能耗盡 512–1023 來源埠，報 `bindresvport: Address already in use`。待認證流量安靜且 TIME_WAIT 消退後再啟一次 ypbind、確認綁定再啟依賴。
- SSSD 套件可自行設 complain；保留 AppArmor enforce，local profile 先在單台驗 compile/restart、新登入和延遲 denial。11/19 各機備份有 `/var/backups/dvlab-nss-<timestamp>/restore`，先確認部署時間和 config 差異。utux Docker socket `216/GROUP` 曾來自 NIS-only group；local root group 建 socket 後以 numeric GID 恢復，核對 owner/mode 和 `/_ping`，中央 GID 改動需同步。

### 2026-11-19 部署與 DROP 驗收

當日五台 client（四台 GPU 加 utux）改由 SSSD 查 NIS。人工 DROP 測試時，已緩存公鑰／群組／sudo 約半秒，未緩存身分離線時查無資料。於 `/etc/sssd/conf.d/90-dvlab-nis-retry.conf` 的 `[domain/dvlab-nis]` 設 `offline_timeout=2`、`offline_timeout_max=2`、`offline_timeout_random_offset=0` 後，解除 DROP 很快回到線上。四台 GPU 的 PAM/NSS 後來改為下節做法，當日離線測試不等於其後 11/22 修補的離線實測。

早期從 complain 恢復 enforce 的本機 policy 記錄在 `/etc/apparmor.d/local/usr.sbin.sssd`，範圍包括 SSSD PID、專用 NSS PAM stack、`/etc/login.defs` 和網卡 type；未覆蓋套件公用 conffile。

## 2026-11-22 NIS 密碼登入修復：SSSD proxy_child、PAM 與 AppArmor

- Zeus 用 `MERGE_PASSWD=true`，NIS 密碼雜湊在 `passwd.byname`。11/19 把 client 的 passwd 順序改為 `files sss systemd` 後，SSSD 線上驗證無法取得 NIS 雜湊；原有快取仍可能驗過舊密碼。
- 20:31 加回末尾的 `nis` 並透過 systemd 設 `SSS_NSS_USE_MEMCACHE=NO`。21:00 valkyrie 登入仍失敗，AppArmor 擋了 `proxy_child` 執行 `/usr/sbin/unix_chkpwd`，另有 `loginuid` 讀取拒絕。21:02 改用 `/etc/sssd/conf.d/20-no-passwd-memcache.conf` 的 `[nss]`、`memcache_size_passwd = 0`，移除上述 systemd drop-in。這一步只處理 NSS 記憶體快取，四台仍實測登入失敗。
- SSSD 2.12.0 的 `proxy_child` 啟動時呼叫 `clearenv()`，且以 `allow_sss_loop=true` 初始化，沒有補設 `_SSS_LOOPS=NO`。在 valkyrie 用相同 AppArmor profile 與 sssd UID/GID 驗證，空環境讀到 `x`，補上 `_SSS_LOOPS=NO` 才取得和 NIS 一致的雜湊。未見 NIS 拒絕紀錄。若要單獨查 NIS，對 `getent -s` 指定 passwd 資料庫和 nis 服務（用冒號連接）；`getent -s nis` 也會覆寫 hosts 查詢，可能誤報 RPC Unknown host。比較前先在主機端遮蔽雜湊，見 [[nis-account-query-safety]]。
- valkyrie 一度試過將獨立的 NSS 設定用 `BindReadOnlyPaths` 套給 SSSD，但 AppArmor 在該 mount namespace 拒絕了 `/run/systemd/notify` 等 socket 存取，已回滾。
- **正式部署**（valkyrie、athena、cthulhu、mazu）：保留 `files sss systemd nis` 和 passwd 記憶體快取停用設定。`/etc/sssd/conf.d/30-dvlab-nis-pam.conf` 指定 `[domain/dvlab-nis]` 的 `proxy_pam_target = sssd-dvlab-nis`。`/etc/pam.d/sssd-dvlab-nis` 在 auth 和 account 都先呼叫 `pam_dvlab_nis_nss.so`，再 include 套件原有的 `sssd-shadowutils`。模組只在專用 PAM service 設 `_SSS_LOOPS=NO`，由原有 `pam_unix` 驗證密碼。
- 模組位於 `/usr/lib/x86_64-linux-gnu/security/pam_dvlab_nis_nss.so`，C 原始碼放在各 client 的 `/usr/local/src/dvlab-nis/pam_dvlab_nis_nss.c`。AppArmor 維持 enforce；`/etc/apparmor.d/local/usr.sbin.sssd` 只新增 `/etc/pam.d/sssd-dvlab-nis r,`。重新部署時先載入專用 PAM 檔的讀取規則，再切換 SSSD PAM target 並重啟；回滾時先撤掉 target override 並重啟，再移除專用檔案和規則。
- **實測**（22:10 台北時間）：四台以同一臨時 NIS 帳號各走一次 SSH 密碼登入、在 zeus 重設後用新密碼登入、再試舊密碼。前兩次均有 `pam_sss` 成功和 `Accepted password`，舊密碼均被拒絕。無新增 `unix_chkpwd` 拒絕。四台公鑰、一般 NSS 查詢、SSSD 設定檢查均通過。臨時帳號已移除，四台均確認查不到。
- 既有七天認證快取政策保留。在新密碼首次驗證成功前，舊快取密碼仍可能依原政策被接受。本輪未測 NIS 離線行為，尚無同學本人重登紀錄，`New Server Setup(private)` 仍暫停更新。

原始碼參考：[proxy_child](https://github.com/SSSD/sssd/blob/2.12.0/src/providers/proxy/proxy_child.c#L485-L537)、[server_setup](https://github.com/SSSD/sssd/blob/2.12.0/src/util/server.c#L456-L518)。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/hapi-fleet/ubuntu-upgrade-recovery-runbook.md)
