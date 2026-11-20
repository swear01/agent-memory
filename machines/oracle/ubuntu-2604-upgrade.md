---
title: oracle 升級到 Ubuntu 26.04 後的 sudo-rs、iptables 與救援路徑
scope: machines/oracle
tags: [oracle, ubuntu, sudo-rs, iptables, netfilter-persistent, oracle-cloud-agent, tailscale, hapi, security]
status: active
created: 2026-11-21
updated: 2026-11-21
---

2026-11-20 把 oracle（hostname `swever`）從 Ubuntu 24.04 升級到 26.04.1 LTS，內核 `7.0.0-1016-oracle`。

## 起因：裝了內核但沒重啟

這台機器開機 98 天，一直跑 `6.17.0-1019`。Ubuntu 已不再維護 linux-oracle-6.17 這條線（tracker 標成 "Ignored end of life"），所以它收不到 CVE-2026-98432（TCP RCE，CVSS 9.8）的修補。其實修好的 `7.0.0-1016` 早就被 unattended-upgrades 裝好了，只差一次重啟。unattended-upgrades 沒開 `Automatic-Reboot`，新內核不會自動生效，要定期看 `/var/run/reboot-required`。

## 26.04 的 sudo-rs 不接受 Oracle Cloud Agent 的 sudoers

- 26.04 的 `sudo` 預設是 sudo-rs（Rust 重寫）。原版 sudo 改名為 `/usr/bin/sudo.ws`，兩者由 `update-alternatives` 切換。
- oracle-cloud-agent snap 的 install/post-refresh hook 會寫 `/etc/sudoers.d/100-oracle-cloud-agent-users`，裡面有 `/bin/systemctl * unified-monitoring-agent.service` 這種參數通配符。sudo-rs 拒絕這種寫法（`wildcards are not allowed in command arguments`），所以每次 sudo 都會印出警告。實際權限沒有少：同一個檔案的 `WLP_SYSTEMD` 已經給了完整的 systemctl。
- agent 升到 1.63.0-9（snap track `latest/stable/ubuntu-26.04`）後還是同樣寫法。手改檔案會在下次 refresh 被覆蓋。
- 採用的做法：`sudo update-alternatives --set sudo /usr/bin/sudo.ws`，之後 `visudo -c` 全部 parsed OK。等 Oracle 修好後，可以用 `update-alternatives --auto sudo` 切回 sudo-rs。
- 這個 snap 的 refresh 是 `hold: forever`，要更新得手動 `sudo snap refresh oracle-cloud-agent`。

## release upgrade 會重存 iptables 規則

- `do-release-upgrade` 過程中，`/etc/iptables/rules.v4` 和 `rules.v6` 被重新存檔，Docker 和 Tailscale 在執行時加的 chain 也被存進去，INPUT 還多了一整組重複規則。已從升級前的 `/etc` 備份恢復乾淨版本，升級時的快照留在 `rules.v4.upgrade-snapshot-20261120`。
- 乾淨的 rules.v4：INPUT 只放行 22，最後 REJECT。43005 和 53346 在 2026-11-20 已移除。53346 原本是 hexapharma 的 Vite dev server，被開到了公網。

## 教訓：不要用規格刪重複的 iptables 規則

INPUT 有重複規則時，`iptables -D INPUT <規格>` 刪掉的是第一個符合的那條。結果刪到的是 REJECT 前面那條 22，公網 SSH 馬上被拒絕。要刪就用規則編號，而且只刪第一個 REJECT 之後的規則（那些本來就不會被匹配到）。

## SSH 斷掉時的救援路徑

1. hapi：oracle 的 runner 是主動連 hub 的，不需要 22 port。用 `hapi spawn-peer --machine <oracle-machine-id> --dir <remote-home> ...` 開 session，讓它在機器上執行修復指令。這次就是用這個方法救回來的。
2. Tailscale：oracle 一直在 tailnet 上（`100.126.43.9`），ts-input 排在 REJECT 前面。但前提是 Mac 的 Tailscale 也要開著。這次 Mac 是關的，2026-11-20 已經設成開機自啟動，見 [[tailscale-autostart]]。從 M5 連線用 `ssh ubuntu@100.126.43.9`。
3. OCI：用 `oci compute instance action --action reset` 或 serial console，見 hapi skill 的 fleet-ops。
