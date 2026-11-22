---
title: DV Lab 機群只有 sshd 內建 PerSourcePenalties；登入失敗先看 journal
scope: machines/hapi-fleet
tags: [ssh, fail2ban, nis, login, troubleshooting]
status: active
created: 2026-11-22
updated: 2026-11-22
---

# DV Lab 機群只有 sshd 內建 PerSourcePenalties

2026-11-22 實查 zeus／valkyrie／athena／cthulhu／mazu：fail2ban 未安裝，sshguard、crowdsec、
denyhosts 皆 inactive，PAM 無 faillock／tally，`/etc/hosts.deny` 空，iptables 無針對 SSH 的 DROP。
但五台都是 Ubuntu 26.04／OpenSSH 10.2，預設啟用 `PerSourcePenalties`（authfail:5 noauth:1
grace-exceeded:10 min:15 max:600，未設 exempt list）：同一來源 IP 累積 ≥15 秒 penalty 就短暫拒絕連線，
log 為 `srclimit_penalise` / `drop connection ... penalty`。實際觸發幾乎都是 exceeded LoginGraceTime，
每次約 20 秒；是 per-host、per-IP，不經 NIS 同步。學生登不進去通常仍是密碼錯。
2026-11-22 b09901086 經 192.168.1.1 在 valkyrie 被擋過一次約 20 秒，同時擋到所有經 NAT 進來的人。

- 判斷：`sudo journalctl -t sshd -t sshd-session --since today | grep <user>`。每次都回
  `Failed password` 表示 sshd 有在驗密碼、沒被擋（被 ban 會是連線 timeout／refused）。
- 不知道帳號時：彙整 `Failed password for <user>`（排除 `invalid user`），再用
  `getent passwd | cut -d: -f1,3,5` 對中文姓名（GECOS）。不要印整筆 passwd，見 [[nis-account-query-safety]]。
- 帳號狀態只在 zeus `/etc/shadow` 用 awk 判斷 `!` 前綴與第 3/5/8 欄，不輸出 hash。
- 部分連線經 NAT，log 來源是 `192.168.1.1`／`192.168.50.1`（共用 IP）。PerSourcePenalties 應設
  `PerSourcePenaltyExemptList`，fail2ban 應設 `ignoreip`，否則一人打錯會擋到所有經 NAT 的人。
- 7 天內外部掃描極少（每台 invalid user 0–3 次），登入 99% 為 publickey；pam_faillock 可被拿學號鎖人，不建議。
- NIS（`MERGE_PASSWD=true`）讓帳密全機群共用，改密碼在 zeus 做後 `make -C /var/yp`；
  但封鎖（若有）屬各主機的防火牆，不會經 NIS 同步。

## 幫學生重設 NIS 密碼（2026-11-22 實作過）
- zeus：`printf 'user:pw\n' | sudo chpasswd` 後 `sudo make -s -C /var/yp`（YPPWDDIR=/etc、MERGE_PASSWD=true），產生 yescrypt。
- 驗證不印 hash：zeus `/etc/shadow` 第 2 欄與各 client `ypmatch user passwd.byname | cut -d: -f2` 各取 sha256 前 12 碼比對。
- 密碼在單一 shell 指令內生成、餵 chpasswd、組信件後 pipe 給 `himalaya -a ntu message send --no-save`，不落地、不輸出；學號帳號信箱為 `<學號>@ntu.edu.tw`。
- zeus 的 rpc.yppasswdd 有在跑，學生可自行用 `yppasswd` 改密碼。

## 2026-11-22 已套用的 penalty 設定（五台）
- `/etc/ssh/sshd_config.d/50-persource-penalties.conf`：
  `PerSourcePenaltyExemptList 140.112.0.0/16,192.168.0.0/16`、`PerSourcePenalties min:30`（其餘維持預設 authfail:5、max:600）。
- 語意：penalty 以「連線」計、隨時間衰減。人手打密碼（每條連線 ~20 秒）累積不到 min，不會被擋；
  bot 快速重連約 10 次失敗後被擋，最長 10 分鐘。這是使用者選的「失敗約 10 次才擋、校內不擋」。
- zeus 的 sshd_config 是舊版、原本沒有 `Include`，已在第 1 行加上（備份 `/etc/ssh/sshd_config.bak-20261122`）。
  之後在 zeus 改 sshd 設定可以用 drop-in 了。
- 檢查：`sudo sshd -T | grep ^persourcepenalt`。
