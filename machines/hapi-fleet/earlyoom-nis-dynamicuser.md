---
title: earlyoom 已裝在四台；NIS 主機要關掉 DynamicUser，不然 systemd 啟動卡死
scope: machines/hapi-fleet
tags: [earlyoom, systemd, nis, oom, fleet]
status: active
created: 2026-11-04
updated: 2026-11-04
---

2026-11-04 經使用者同意，用 `su_<host>` + `sudo -n` 在 valkyrie、cthulhu、mazu、zeus 裝 earlyoom，並停用 systemd-oomd。起因是 valkyrie 記憶體耗盡整台卡死，systemd-oomd 沒擋住。athena 送修中，尚未安裝。

- 設定 `/etc/default/earlyoom`：`-m 5 -s 100 -r 3600`；`--avoid` sshd、systemd*、dockerd、containerd*、tailscaled、hapi、Xorg；`--prefer` abc、gpuls*、dc_shell*、genus*。
- 坑：valkyrie、cthulhu、mazu 的 `/etc/nsswitch.conf` 的 passwd 那行是 `files nis sss systemd`。Ubuntu 的 earlyoom unit 用 `DynamicUser=true` 加 `PrivateNetwork=true`。systemd 在 exec 前分配動態使用者時會查 NSS，NIS 要連 本機 rpcbind（port 111），但在空的 network namespace 裡 SYN 永遠沒回應，行程一直停在 `(earlyoom)`，`systemctl restart` 要等 90 秒逾時。`systemctl is-active` 仍顯示 active，容易誤判。
- 解法：drop-in `/etc/systemd/system/earlyoom.service.d/nis.conf` 寫 `[Service]` `DynamicUser=no`，`daemon-reload` 後 `systemctl kill -s KILL earlyoom` 再 restart。zeus 沒有 NIS，原版 unit 就正常。
- 驗證：`/proc/$(systemctl show -p MainPID --value earlyoom)/cmdline` 要是 `/usr/bin/earlyoom ...`，journal 有 `mem avail:` 行。
