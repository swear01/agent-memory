---
title: ADFP Chromium key exception and FortiClient recovery boundary
scope: projects/ntu-cvsd-adfp
status: launcher-verified; ADFP-live-test-blocked-by-VPN
updated: 2026-10-14
---

# Approved dedicated-browser exception

- The ADFP server certificate was self-signed and lacked SAN. Importing it alone does not resolve Chromium's separate hostname verification; do not repeat the earlier host-scoped Keychain trust attempt as a proven fix.
- On 2026-10-13 the user explicitly approved a public-key exception in the dedicated ADFP Chromium profile, after being told it is not restricted to one IP. This supersedes the earlier manual-certificate-warning handoff only for this approved key exception.
- The private course repository's PR #12 merged `tools/adfp-chromium.command`, its usage/rollback documentation and eight mocked VPN/process/launch checks. The launcher requires the existing VPN, uses the existing dedicated profile and loopback CDP, and passes `--ignore-certificate-errors-spki-list` with the approved fixed SPKI. Read the key from that verified launcher; never automatically replace it with a key fetched from an unverified connection.
- This exception accepts certificate chains containing the matching public key regardless of hostname or certificate validation errors. It is broader than an exact leaf-certificate fingerprint exception. Keep this browser profile dedicated to ADFP; other browser profiles and macOS trust settings were not changed. Directly opening Chromium does not persist the startup flag across future launches.
- Installed Chromium 150 was tested against a local self-signed HTTPS server: no matching key failed, matching key loaded the expected body, and the actual ADFP key exception rejected the unrelated test key. The dedicated browser was launched with the approved flag and its process arguments checked. These checks do not prove ADFP itself loads without a warning.
- To undo the exception, remove the flag and restart the dedicated browser. A new server key requires a new verification and trust decision.

## FortiClient blocker observed during restart

- The already-modified FortiClient `Contents/Info.plist` contained only a URL-scheme array, not an application dictionary. Starting the GUI crashed with `Unable to find helper app`; signature verification also failed. The modification predated this certificate task; its author and broader cause were not established.
- A minimal metadata reconstruction failed signature verification and was rolled back byte-for-byte. Do not retain guessed metadata, re-sign the app, erase browser/VPN settings, or claim the application was restored.
- The existing official online installer downloaded partial payloads but failed with FDS connection resets/timeouts on three servers. No complete original payload was obtained. The task-created installer was stopped and its mounted image detached; VPN remained Disconnected.
- Recovery needs an authentic compatible installer or original signed metadata, followed by GUI/VPN verification. ADFP's live no-warning test remains unverified. Recheck current state before resuming; do not treat this dated blocker as permanent or restart the old failed download queue automatically.
- Avoid dumping full `scutil --nc status`: it can include a VPN session secret. When only connectivity is needed, output only the first status line. Credentials, session values and raw private logs must stay out of memory and Git.
