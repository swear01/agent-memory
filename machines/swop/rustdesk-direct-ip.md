---
title: swop RustDesk relay timeout and Tailscale direct IP access
scope: machines/swop
machine: swop
status: active
updated: 2026-10-30
---

# swop (Swear01_PC) RustDesk

- RustDesk ID 1998038884, service `RustDesk` runs as LocalService; service config is `C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\config\RustDesk2.toml`, logs under `...\RustDesk\log\server`. Readable/editable via `ssh swop` (that session is elevated); the Grok local-exec shell is not.
- Uses public rendezvous rs-ny.rustdesk.com, permanent-password verification, 2FA on.
- 2026-10-30 failure: hole punch from NTU (140.112.171.141) failed, fallback relay `ovh-west2.rustdesk.com` → `Failed to create relay connection ... deadline has elapsed`. Public relay flakiness, not auth.
- Fix: `direct-server = 'Y'` in service config `[options]` (backup `.bak-20261030`), service restarted; listens on TCP 21118. Firewall: rule "RustDesk direct 21118 block non-Tailscale" blocks all except 100.64.0.0/10, plus an allow rule for 100.64.0.0/10. Connect by entering Tailscale IP `100.69.9.16` in RustDesk.
