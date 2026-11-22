---
title: DV Lab 機群沒有 SSH 封鎖機制；登入失敗先看 journal
scope: machines/hapi-fleet
tags: [ssh, fail2ban, nis, login, troubleshooting]
status: active
created: 2026-11-22
updated: 2026-11-22
---

# DV Lab 機群沒有 SSH 封鎖機制

2026-11-22 實查 zeus／valkyrie／athena／cthulhu／mazu：fail2ban 未安裝，sshguard、crowdsec、
denyhosts 皆 inactive，PAM 無 faillock／tally，`/etc/hosts.deny` 空，iptables 無針對 SSH 的 DROP。
所以「帳號被 ban」不成立；學生登不進去通常是密碼錯。

- 判斷：`sudo journalctl -t sshd -t sshd-session --since today | grep <user>`。每次都回
  `Failed password` 表示 sshd 有在驗密碼、沒被擋（被 ban 會是連線 timeout／refused）。
- 不知道帳號時：彙整 `Failed password for <user>`（排除 `invalid user`），再用
  `getent passwd | cut -d: -f1,3,5` 對中文姓名（GECOS）。不要印整筆 passwd，見 [[nis-account-query-safety]]。
- 帳號狀態只在 zeus `/etc/shadow` 用 awk 判斷 `!` 前綴與第 3/5/8 欄，不輸出 hash。
- 外部連線經 router NAT 進來，log 來源常是 `192.168.1.1`。若日後裝 fail2ban，必須把
  192.168.1.0/24 列入 `ignoreip`，否則一個人打錯密碼會把所有從外面連進來的人一起 ban 掉。
- NIS（`MERGE_PASSWD=true`）讓帳密全機群共用，改密碼在 zeus 做後 `make -C /var/yp`；
  但封鎖（若有）屬各主機的防火牆，不會經 NIS 同步。
