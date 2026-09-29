---
title: zeus systemd unit for DVLab Minecraft Discord bot
scope: machines/zeus
status: active
updated: 2026-09-29
---

# zeus：DVLab Minecraft Discord bot systemd

- Unit name: `dvlab-mc-discord-bot.service`
- State: enabled and active (running) as of 2026-09-29
- Working directory / code: `<zeus-home>/dvlab-mc-discord-bot` under account **swear01**
- Process user: `swear01`
- Install privilege: **swear01 cannot sudo** on zeus; unit was installed and enabled with **swear02** passwordless sudo (`su_zeus` style access)
- Env: Discord bot credential in zeus home `.env` (mode 600); keep out of git
- Purpose: Discord presence + slash `/status` `/players` `/ip` for Vault Hunters (utux), polling `140.112.171.142:25565`
- Restart after pull/code change: `systemctl restart dvlab-mc-discord-bot` (via swear02 sudo)
- Full product note: `projects/dvlab-mis/minecraft-discord-bot-plan.md`
