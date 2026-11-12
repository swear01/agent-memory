---
title: earlyoom 已裝在四台；NIS 主機要關掉 DynamicUser，不然 systemd 啟動卡死
scope: machines/hapi-fleet
tags: [earlyoom, systemd, nis, oom, fleet]
status: active
created: 2026-11-04
updated: 2026-11-12
---

2026-11-04 經使用者同意，用 `su_<host>` + `sudo -n` 在 valkyrie、cthulhu、mazu、zeus 裝 earlyoom，並停用 systemd-oomd。起因是 valkyrie 記憶體耗盡整台卡死，systemd-oomd 沒擋住。athena 於 2026-11-12 回機後補裝，設定相同。

- 設定 `/etc/default/earlyoom`：`-m 5 -s 100 -r 3600`；`--avoid` sshd、systemd*、dockerd、containerd*、tailscaled、hapi、Xorg；`--prefer` abc、gpuls*、dc_shell*、genus*。
- 坑：valkyrie、cthulhu、mazu 的 `/etc/nsswitch.conf` 的 passwd 那行是 `files nis sss systemd`。Ubuntu 的 earlyoom unit 用 `DynamicUser=true` 加 `PrivateNetwork=true`。systemd 在 exec 前分配動態使用者時會查 NSS，NIS 要連 本機 rpcbind（port 111），但在空的 network namespace 裡 SYN 永遠沒回應，行程一直停在 `(earlyoom)`，`systemctl restart` 要等 90 秒逾時。`systemctl is-active` 仍顯示 active，容易誤判。
- 解法：drop-in `/etc/systemd/system/earlyoom.service.d/nis.conf` 寫 `[Service]` `DynamicUser=no`，`daemon-reload` 後 `systemctl kill -s KILL earlyoom` 再 restart。zeus 沒有 NIS，原版 unit 就正常。
- 驗證：`/proc/$(systemctl show -p MainPID --value earlyoom)/cmdline` 要是 `/usr/bin/earlyoom ...`，journal 有 `mem avail:` 行。
- 2026-11-12 新發現：`user@.service` 對 `systemd-oomd.service` 有 Wants+After。NSS 把 `systemd` 放最後時，oomd 啟動也會卡住（推論同為 sandbox 內查 NIS，未用 strace 確認），每個新 user manager 要等 oomd 90 秒逾時，SSH 登入約 2 分鐘（Athena 實測）。只 `disable` 不夠，因為 Wants 仍會把它拉起來。
- 處理（使用者同意）：athena、mazu、cthulhu、valkyrie 都 `apt-get remove systemd-oomd`（只移除它本身；gnome-settings-daemon、ubuntu-desktop 只是 Recommends），並保留 `systemctl mask systemd-oomd.service systemd-oomd.socket`，防止日後 desktop meta-package 又裝回來。四台 SSH 登入 0.3–0.5 秒，earlyoom active。zeus 本來就沒裝這個套件。
