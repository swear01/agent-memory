---
title: Bitwarden CLI against the user's self-hosted server
scope: tools/bitwarden
machine: swairM5
status: active
updated: 2026-10-30
---

# Bitwarden CLI with user's self-hosted server

- Server: https://swever.tailbfa9de.ts.net (Tailscale only). Desktop app config at ~/Library/Containers/com.bitwarden.desktop/Data/Library/Application Support/Bitwarden/data.json.
- `bw` (Homebrew, /opt/homebrew/bin/bw) on SwairM5: `bw config server https://swever.tailbfa9de.ts.net` must be set before `bw login`, else it hits bitwarden.com.
- User must run `bw login` and `bw unlock --raw > ~/.bw_session && chmod 600 ~/.bw_session` (master password). If unlock is run before login the session file is empty.
- Agent then: `BW_SESSION=$(cat ~/.bw_session)`, build item JSON in python (never echo secrets), `bw encode | bw create item`, `bw create attachment --file … --itemid …`, verify by `bw get attachment` + sha256, then `bw lock` and `rm -P ~/.bw_session`.
- zsh: unmatched glob in `ls ~/x-*` aborts the command under `set -e`; use `ls … 2>&1 || true`.
