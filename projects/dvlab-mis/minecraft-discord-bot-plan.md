---
title: DVLab Minecraft Discord Bot implementation plan
scope: projects/dvlab-mis
status: live
updated: 2026-09-29
---

# DVLab Minecraft Discord Bot（已上線）

## Goal
Discord bot on **zeus** that shows Vault Hunters (utux, MOTD `dvlab server`) online count in presence, and answers slash commands for server info and online player IDs. Owned under Google account **dvlabdvlab** via Discord Developer Portal.

## Decisions
- Application: reuse **DVLAB Bot** `1550481869618024468` (was MIS/EDA-oriented; little/no live code yet)
- Host: **zeus** (lab service machine)
- Game: Vault Hunters Remastered on **utux**; no RCON; FIFO `vault-hunters-console` for console
- Data: public Minecraft **server-list ping** first; **FIFO `list`** fallback if player names missing (v1 uses ping sample only)
- UX: Discord activity type **Watching**; activity name `minecraft N/8 players` (UI shows as Watching minecraft N/8 players)
- Slash only: `/status`, `/players`, `/ip`
- Guild: `NTU DVLab` `1155808217872466041`, mainly `#⛏⛏⛏`

## Live deployment (2026-09-29)
- Code path on zeus: `<zeus-home>/dvlab-mc-discord-bot` (Node, discord.js + minecraft-server-util + dotenv)
- Bot identity: `DVLAB Bot#8714`
- Poll target: `140.112.171.142:25565` (inari website-host forward; also answers on `140.112.171.143:25565` utux 1:1 NAT)
- Discord bot token stored in zeus home `.env` mode `600` (never commit; never paste into chat)
- Persistence: systemd unit `dvlab-mc-discord-bot.service`, **enabled** + **active**; unit installed with **swear02** passwordless sudo because **swear01 cannot sudo** on zeus; process `User=swear01`
- Guild slash commands registered for `1155808217872466041`
- Box NTU VPN reaches campus sites (www/cool) but times out to `140.112.171.x`; deploy/ops via Mac SSH (`Host zeus`)

## Ops cheatsheet (zeus)
- Status: `systemctl status dvlab-mc-discord-bot`
- Logs: `journalctl -u dvlab-mc-discord-bot -f`
- Restart: `sudo systemctl restart dvlab-mc-discord-bot` (needs swear02 or equivalent sudo)
- After code change under `<zeus-home>/dvlab-mc-discord-bot`: install deps if needed, then restart the unit

## Out of scope (v1)
- Chat bridge, account linking, RCON, Paper plugins, LLM natural language, MIS/EDA commands
- FIFO ProxyJump fallback (add only if ping stops returning player samples)

## Related notes
- Vault Hunters ops: `projects/dvlab-mis/vault-hunters-remastered-operations.md`
- Bot branding: `projects/dvlab-mis/discord-bot-identity-and-assets.md`
- Zeus unit note: `machines/zeus/dvlab-mc-discord-bot-systemd.md`
