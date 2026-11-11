---
title: mazu 免 root 的 GSAT DGX H100 VPN 轉接
scope: machines/mazu
machine: mazu
status: active
updated: 2026-11-11
---

- 實驗室主機（zeus/valkyrie/cthulhu/mazu）經由 mazu 連到重點科技研究學院的 DGX H100（192.168.163.102，只能透過 FortiGate SSL VPN 140.112.207.229:11443 連）。
- 做法：openconnect `--protocol=fortinet` 加 `--script-tun` 搭配 ocproxy，全程在使用者空間，不建 tun、不改路由、不需要 sudo。Ubuntu 26.04 的 deb 用 `apt-get download` 下載，再用 `dpkg -x` 解到 `<remote-home>/opt/dgxvpn/root`，靠 `LD_LIBRARY_PATH` 補依賴：libopenconnect5、libstoken1t64、libpskc0t64、libtomcrypt1、libtommath1、libxmlsec1-1、libxmlsec1-openssl1、ocproxy、vpnc-scripts。
- 服務：`systemctl --user` 的 `dgxvpn.service`（Restart=always，RestartSec=60，mazu 已開 linger），執行 `<remote-home>/opt/dgxvpn/run.sh`。在 mazu 上開 SOCKS5 127.0.0.1:1080，並把 127.0.0.1:2222 轉到 DGX:22。帳密放在 `<remote-home>/.config/dgxvpn/{user,pass}`（0600），用 `vault run` 寫入，不印出。伺服器憑證用 `--servercert pin-sha256` 固定，Mac 端 openssl 交叉確認過指紋相同。
- 閘道不允許 reconnect-after-drop，idle timeout 60 分鐘，session 8 小時到期；斷線後 openconnect 會退出，由 systemd 重啟並重新登入。
- 實驗室主機共用 NFS home，所以只要在共用的 `~/.ssh/config` 加一個 `Host dgx`：在 mazu 上 `nc 127.0.0.1 2222`，在其他主機上 `ssh -W` 經過 mazu。DGX 帳號是 VPN 帳號，sshd 接受 publickey 和 password；要先由使用者自己跑一次 `ssh-copy-id dgx`，把共用的 fleet 金鑰裝上去。
- 未驗證：Mac 的 DGX VPN（[[dgx-vpn]] 在 machines/swairM5）和 mazu 用同一個帳號同時連線，閘道會不會互踢。
