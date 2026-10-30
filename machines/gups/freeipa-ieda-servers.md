---
title: ieda1/ieda2 lab servers: FreeIPA, network, hardening
scope: machines/gups
machine: gups
status: active
updated: 2026-10-30
---

# ieda1 / ieda2 lab servers (FreeIPA) — 2026-10-30

- ieda1.gups.ntu.edu.tw = old `gups` / intl.gups.ntu.edu.tw (public 140.112.186.18, LAN 192.168.100.254 on enp70s0). FreeIPA 4.9.13 server (idm:DL1), realm GUPS.NTU.EDU.TW, **no integrated DNS** — names via /etc/hosts LAN entries.
- ieda2.gups.ntu.edu.tw = second Rocky 8.10 box (LAN 192.168.100.1, public 140.112.186.23/24 gw .254 on eno1 via NetworkManager; enp70s0 `never-default yes`). IPA client.
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
