---
title: DVLab Minecraft Discord Bot implementation plan
scope: projects/dvlab-mis
status: approved
updated: 2026-09-29
---

# DVLab Minecraft Discord Bot（已核准規格）

## Goal
Discord bot on **zeus** that shows Vault Hunters (utux, MOTD `dvlab server`) online count in presence, and answers slash commands for server info and online player IDs. Owned under Google account **dvlabdvlab** via Discord Developer Portal.

## Decisions
- Application: reuse **DVLAB Bot** `1550481869618024468` (was MIS/EDA-oriented; little/no live code yet)
- Host: **zeus** (lab service machine)
- Game: Vault Hunters Remastered on **utux**; no RCON; FIFO `vault-hunters-console` for console
- Data: public Minecraft **server-list ping** first; **FIFO `list`** fallback if player names missing
- UX: presence `Watching N/M players`; slash only: `/status`, `/players`, optional `/ip`
- Guild: `NTU DVLab` `1155808217872466041`, mainly `#⛏⛏⛏`

## Implementation outline
1. **Portal**: confirm app owned by `dvlabdvlab`; enable Bot + `applications.commands`; copy token (secret on zeus only); invite with bot + applications.commands
2. **Reachability test from zeus**: `mcstatus <join-host:port> status` (or equivalent) — confirm online count and whether sample player list is returned
3. **Choose codebase**: prefer fork/config of [PetyXbron/minecraft-bot](https://github.com/PetyXbron/minecraft-bot) or thin discord.js/discord.py + mcstatus; config IP/port from network manual (not committed)
4. **Deploy on zeus**: systemd unit, Node 18+ or Python 3.11+, env file for `DISCORD_TOKEN` + MC host/port; restart on failure
5. **Commands**: register guild slash commands; `/status` embed (online, MOTD, version, latency); `/players` names; `/ip` if approved
6. **Presence loop**: poll every ~30–60s; respect Discord rate limits
7. **FIFO fallback** (only if ping lacks names): from zeus SSH ProxyJump to utux, run `sudo /usr/local/sbin/vault-hunters-console 'list'`, parse journal/log; keep credentials out of repo
8. **Docs**: update agent-memory + Drive runbook when live

## Out of scope (v1)
- Chat bridge, account linking, RCON, Paper plugins, LLM natural language, MIS/EDA commands

## Blockers / need from user
- Confirm `dvlabdvlab` can open the Application (or transfer)
- Bot token via secure channel (never chat paste)
- Exact public join host:port as seen from zeus
- zeus SSH access for deploy (or approve assistant deploy path)
