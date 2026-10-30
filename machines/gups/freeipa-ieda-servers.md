---
title: ieda1/ieda2 lab servers: FreeIPA, network, hardening
scope: machines/gups
machine: gups
status: active
updated: 2026-10-30
---

# ieda1 / ieda2 lab servers (FreeIPA) — 2026-10-30

- ieda1.gups.ntu.edu.tw = old `gups` / intl.gups.ntu.edu.tw (public 140.112.186.18, LAN 192.168.100.254 on enp70s0). FreeIPA 4.9.13 server (idm:DL1), realm GUPS.NTU.EDU.TW, **no integrated DNS** — names via /etc/hosts LAN entries.
- ieda2.gups.ntu.edu.tw (planned public DNS intl2.gups.ntu.edu.tw) = second Rocky 8.10 box (LAN 192.168.100.1, public 140.112.186.23/24 gw .254 on eno1 via NetworkManager; enp70s0 `never-default yes`). IPA client.
- IPA ports only in firewalld zone `internal` (LAN). Public zone: only 22. xrdp (3389) and cockpit disabled on both.
- fail2ban 1.0.2 both: systemd backend, firewallcmd-rich-rules, bantime 1h increment, findtime 10m, maxretry 5, ignoreip includes LAN, Tailscale 100.64/10.
- ieda2 root: key-only (`PermitRootLogin prohibit-password`); keys = SwairM5 + gups root ed25519. SwairM5 `ssh ieda2` → 140.112.186.23.
- Secrets: ieda1:/root/ipa-credentials.txt (600), /root/cacert.p12; copy in user's self-hosted Bitwarden item "ieda1 FreeIPA" (admin login + Directory Manager hidden field + cacert.p12 attachment). Never print them.
- Backup: /etc/cron.d/ipa-backup Sun 04:00 `ipa-backup --data --online` → rsync to ieda2:/root/ipa-backups/.
- Group `pdk_approved` GID 1024 = local PDK_approved (IPA lowercases group names). sudo only for IPA `admins` (`sudorule-add-user --groups=admins`; no sudorule-add-group). HBAC allow_all still on.
- Old local student accounts on gups are stale; not migrated. /home not NFS-shared (NDA PDK lives under the home tree, `PDK` dir).
- EDA licenses are remote at TSRI (lshc/lstn/lstc in /etc/hosts — keep them); not hostname-bound, but TSRI must whitelist the server's public IP.

## Gotchas
- `ipa-server-install` fails under umask 077 → run with umask 0022.
- Passing admin password on `ipa-client-install` command line exposes it in `ps`; prefer prompt/file.
- Gateway 140.112.186.254 doesn't answer ping but routes fine.

## Public DNS for TSRI IP registration (2026-10-30)
- TSRI「IP 登錄」只收「大專院校 IP」：IP 的 PTR 名稱須以 `edu.tw` 結尾，且該名稱的 A 正解回同一 IP（TSRI 申請須知附錄 A）。
- 已驗證（Google DoH）：140.112.186.18 ↔ `intl.gups.ntu.edu.tw` 正反解一致，保留此名，不改 intl1。SwairM5 本機 dig 曾錯誤顯示 PTR 為空，查 DNS 用公開 resolver 交叉確認。
- 140.112.186.23 已分配但無 A/PTR；預定名稱 `intl2.gups.ntu.edu.tw`。`intl` 公開名稱沒有任何設定依賴（SSH config 全用 IP；兩台 /etc/hosts 的 intl 別名指 LAN 192.168.100.254）。
- 管理方：`186.112.140.in-addr.arpa` 與 `intl.gups.ntu.edu.tw` 都在計中 DNS（ntuns / dns.ntu.edu.tw），GUPS 不自管 zone；`gups.ntu.edu.tw` 是 CNAME 到 Bunny CDN。
- 申請流程：由單位（GUPS 窗口歐姍姍）填計中「網域名稱申請表」、主管核章送網路組（https://ccnet.ntu.edu.tw/ccnet/domain.php ；DNS 業務窗口見申請表 PDF），學生不能自送。2026-10-30 已寄信請歐姍姍申請 intl2 A+PTR，待回覆。DNS 好了之後再請王信博老師在 TSRI「IP 登錄」上傳 .18、.23。

## Account creation (2026-10-30)
- `ieda1:/root/ipa_add_user.sh <user> <name> <email> [reset]` reads the password on stdin. It does a kinit as admin using the root-only credentials file, runs ipa user-add, adds the user to pdk_approved, and sets the password. It then sets krbPasswordExpiration to now+90d (so there is no forced change at first login) and creates the home with mkhomedir_helper on ieda1. Homes must exist on ieda1 or ieda2's autofs mount fails.
- The Drive script `gups_accounts.sh` (in the GUPS TA folder) calls it over ssh and mails the password via himalaya. `--preview` sends nothing. It was tested end to end with a throwaway account on both servers, and that account was deleted.
- The roughly 20 older student accounts on ieda1 are still local `/etc/passwd` users, not in IPA, so they cannot log in to ieda2. Migrating them is the user's call.
- The handover doc 「GUPS伺服器交接資訊.md」 was rewritten on 2026-10-30. It points to Bitwarden for the IPA secrets; the ieda1 root password line was kept as it was.
