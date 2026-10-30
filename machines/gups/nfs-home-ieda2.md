---
title: ieda1 /home shared to ieda2 over NFS (single home)
date: 2026-10-30
tags: [gups, ieda1, ieda2, nfs, autofs]
---
# ieda1 /home shared to ieda2 over NFS

- ieda1 exports `/home` only to ieda2's LAN address 192.168.100.1: `rw,sync,root_squash,no_subtree_check`, NFSv4 only (`vers3=n` in nfs.conf).
- firewalld: the `nfs` service is opened only in zone `internal` (LAN); the public zone has just ssh and dhcpv6-client.
- ieda2 uses autofs: `/etc/auto.master.d/home.autofs` maps `/home` to `/etc/auto.home`, which holds the wildcard entry
  `* -fstype=nfs4,rw,soft,timeo=50,retrans=2,retry=0,nosuid,nodev ieda1:(home)/&`. SELinux boolean `use_nfs_home_dirs=on`.
- joinet's two homes were merged into ieda1's copy: ieda2-only files were added (rsync --ignore-existing), and .bash_history and known_hosts were combined.
  Backups: the old ieda2 home is in a dated `home.ieda2-old` dir at the root of ieda2 and `/root/home-joinet-ieda2-20261030.tgz` on ieda2; ieda1 /root keeps the pre-merge bash_history and known_hosts.
- joinet is uid 1000 on both servers. ieda2 got a local group PDK_approved (gid 1024, same as ieda1) with joinet as a member, so the PDK is readable. Root is squashed, so ieda2's root cannot read the PDK.
- LAN outage test (ieda2's enp70s0 link down): root login, `/`, df and sssd lookups respond instantly. Access to NFS homes fails with an error after about 120 s instead of hanging forever, and a joinet login shell takes about 3 minutes. Everything recovers automatically once the link is back.
