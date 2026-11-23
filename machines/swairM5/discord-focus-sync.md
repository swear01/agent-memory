---
title: macOS Focus syncs Discord Do Not Disturb through the official Social SDK
scope: machine
machine: swairM5
tags: [macOS, Focus, Discord, Shortcuts, Social SDK, OAuth2, launchd, Live Activities, WidgetRenderer_Activities, WatchPaths]
status: active
updated: 2026-11-23
---

# Discord Focus Sync

On swairM5 (macOS 27.2.1), opening any active macOS Focus sets the Discord user status to Do Not Disturb. Ending Focus restores the earlier Online, Idle, or Invisible status if the watcher still owns the DND change. Existing DND and a different manual status observed on restore are respected. The one-shot helper uses Discord Social SDK 1.11.24259 `SetOnlineStatus()` with OAuth2 PKCE and the presence scope. The signed-in Discord client token is never extracted. The scoped refresh credential is stored in the login Keychain.

## Installed components

- Shortcut `Focus Sync Current Focus` returns the literal text `on` or `off` via Apple's Get Current Focus action.
- `local.focus-sync.sample` runs the shortcut only at load and when launchd `WatchPaths` sees `~/Library/DoNotDisturb/DB/Assertions.json` or its `DB` directory change (no `StartInterval`). It atomically records a successful result in `~/Library/Application Support/Focus Sync/focus-state` and wakes the reconciliation job when the value changes. The original five-second polling plist is kept as `~/Library/Application Support/Focus Sync/local.focus-sync.sample.plist.bak-poll5s`.
- `local.focus-sync.watch` runs a short shell reconciliation on wake or every 15 seconds as fallback. It invokes the SDK only for an unapplied state that needs Discord, then exits. It uses a per-account `owned-status` journal to preserve the pre-DND status across processes. A stale sample is skipped and transient failures are retried with a 20-second same-edge cooldown. Both jobs and support files remain under `~/Library/LaunchAgents` and `~/Library/Application Support/Focus Sync`.
- Use `launchctl print gui/$UID/local.focus-sync.watch`, `launchctl print gui/$UID/local.focus-sync.sample`, and `~/Library/Application Support/Focus Sync/focus-sync.log` to verify operation. For a manual Shortcut test use `shortcuts run 'Focus Sync Current Focus' --output-path <new-output-file> --output-type public.plain-text`; stdout can be empty despite success.

## Verified behavior and limits

- Discord OAuth through the Codex in-app browser and localhost callback succeeded. Both LaunchAgents started under the current login session. A live Shortcut sample switched Discord to DND. An isolated synthetic off sample restored Online, then restarting the sampler reapplied DND from the Mac's actual Focus. The previous resident watcher's graceful shutdown also restored Online. With the on-demand install, a synthetic off sample restored Online in 4.6 seconds and the real on sample reapplied DND and exited in 6.1 seconds. A separate test confirmed DND survived explicit SDK disconnect and process exit. The earlier cold on test took 18 seconds, which the user accepted.
- A full reboot or logout/login and the actual notification sound were not tested. Manual status changes that end back in DND entirely between one-shot invocations cannot be distinguished from owned DND.
- A process run directly from Codex could read the private macOS Focus JSON; the independent LaunchAgent got `Operation not permitted` on the file contents. The signed Shortcut worked from launchd and avoided granting Full Disk Access. It does not open Discord settings or drive the mouse.

## SDK size and alternatives

The official C++ archive is about 752 MiB on disk and contains debug/release and multiple platforms. The original installation used about 71 MiB, including a 70 MiB macOS framework with bundled Krisp voice assets. The current installation uses about 12 MiB: the standalone release dylib from the same archive with only its arm64 slice, plus a size-optimized helper of about 152 KiB. The dylib passed strict signature verification with the original Discord Team ID. The live LaunchAgent was relaunched, applied DND on the real Focus sample, restored Online on a synthetic off sample, and reapplied DND when the sampler resumed. Old and new watcher startup physical-footprint peaks were about 6.5 and 6.7 MiB respectively, so the packaging change alone did not reduce startup RAM. The on-demand SDK process exits between changes, eliminating its idle memory use. The macOS Shortcut now runs only when Focus changes. Discord does not offer a separate smaller C++ download for this configuration.

Discord documents `SetOnlineStatus()` in the [Social SDK](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html). The [public local RPC](https://github.com/discord/discord-api-docs/blob/main/developers/topics/rpc.mdx) exposes Rich Presence activity updates, not the account's DND setting. [Metrolist's merged implementation and test notes](https://github.com/MetrolistGroup/Metrolist/pull/3874) show the official scoped OAuth and DND path in a non-game app.

## 2026-11-23: five-second Shortcut polling overheated the Mac

Each `shortcuts run` posts a Shortcuts Live Activity (the menu bar Shortcuts icon with a check mark). At one run every five seconds, `WidgetRenderer_Activities` spun in a Combine update loop and leaked memory: it grew from 275 MiB to 1.6 GiB at up to 162% CPU, WindowServer ran at 75%, the load average reached 164, swap grew, and macOS showed the "out of application memory" dialog. CPU resource reports were at 04:28 and 09:04, after the 03:26 install. `killall WidgetRenderer_Activities` relieves it temporarily; launchd restarts the renderer.

Fix, verified in both directions: the LaunchAgent cannot read the Focus JSON, but launchd `WatchPaths` still sees it change. Turning Do Not Disturb off and on from the menu bar changed `Assertions.json` within a second, and the sampler ran within a second. Discord returned to Online at 5 seconds and DND at 8 seconds. Polling was removed after the event path proved reliable.

Diagnosis tip: `~/Library/Group Containers/group.com.apple.liveactivitiesd/liveactivitiesd.sql` lists Live Activities. In the `Activities` table, the base64 `attributesData` inside `request` holds the Shortcut title; the `Assertions` table showed tens of thousands of foreground assertions from the spinning renderer.
