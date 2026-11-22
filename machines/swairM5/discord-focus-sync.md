---
title: macOS Focus syncs Discord Do Not Disturb through the official Social SDK
scope: machine
machine: swairM5
tags: [macOS, Focus, Discord, Shortcuts, Social SDK, OAuth2, launchd]
status: active
updated: 2026-11-23
---

# Discord Focus Sync

On swairM5 (macOS 27.2.1), opening any active macOS Focus sets the Discord user status to Do Not Disturb. Ending Focus restores the earlier Online, Idle, or Invisible status if the watcher still owns the DND change. Existing DND and manual Discord status changes are respected. The watcher uses Discord Social SDK 1.11.24259 `SetOnlineStatus()` with OAuth2 PKCE and the presence scope. The signed-in Discord client token is never extracted. The scoped refresh credential is stored in the login Keychain.

## Installed components

- Shortcut `Focus Sync Current Focus` returns the literal text `on` or `off` via Apple's Get Current Focus action.
- `local.focus-sync.sample` runs the shortcut under launchd about every five seconds and atomically records a successful result in `~/Library/Application Support/Focus Sync/focus-state`.
- `local.focus-sync.watch` reads that state and applies Discord status. The program and official release dylib are in `~/Library/Application Support/Focus Sync`. The two job definitions are in `~/Library/LaunchAgents`.
- Use `launchctl print gui/$UID/local.focus-sync.watch`, `launchctl print gui/$UID/local.focus-sync.sample`, and `~/Library/Application Support/Focus Sync/focus-sync.log` to verify operation. For a manual Shortcut test use `shortcuts run 'Focus Sync Current Focus' --output-path <new-output-file> --output-type public.plain-text`; stdout can be empty despite success.

## Verified behavior and limits

- Discord OAuth through the Codex in-app browser and localhost callback succeeded. Both LaunchAgents started under the current login session. A live Shortcut sample switched Discord to DND. An isolated synthetic off sample restored Online, then restarting the sampler reapplied DND from the Mac's actual Focus. Graceful watcher shutdown also restored Online.
- A full reboot or logout/login and the actual notification sound were not tested. If the watcher is forcibly killed while it owns DND, its in-memory prior status can be lost.
- A process run directly from Codex could read the private macOS Focus JSON; the independent LaunchAgent got `Operation not permitted` on the file contents. The signed Shortcut worked from launchd and avoided granting Full Disk Access. It does not open Discord settings or drive the mouse.

## SDK size and alternatives

The official C++ archive is about 752 MiB on disk and contains debug/release and multiple platforms. The original installation used about 71 MiB, including a 70 MiB macOS framework with bundled Krisp voice assets. The current installation uses about 12 MiB: the standalone release dylib from the same archive with only its arm64 slice, plus a size-optimized helper of about 144 KiB. The dylib passed strict signature verification with the original Discord Team ID. The live LaunchAgent was relaunched, applied DND on the real Focus sample, restored Online on a synthetic off sample, and reapplied DND when the sampler resumed. Old and new watcher startup physical-footprint peaks were about 6.5 and 6.7 MiB respectively, so this change reduces disk usage without a measured RAM improvement. Discord does not offer a separate smaller C++ download for this configuration.

Discord documents `SetOnlineStatus()` in the [Social SDK](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html). The [public local RPC](https://github.com/discord/discord-api-docs/blob/main/developers/topics/rpc.mdx) exposes Rich Presence activity updates, not the account's DND setting. [Metrolist's merged implementation and test notes](https://github.com/MetrolistGroup/Metrolist/pull/3874) show the official scoped OAuth and DND path in a non-game app.
