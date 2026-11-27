---
title: macOS Focus syncs Discord Do Not Disturb through the official Social SDK
scope: machine
machine: swairM5
tags: [macOS, Focus, Discord, Shortcuts, Social SDK, OAuth2, launchd, Live Activities, WidgetRenderer_Activities, WatchPaths]
status: active
updated: 2026-11-23
---

# Discord Focus Sync

- Focus sets Discord to Do Not Disturb. On exit, restore the earlier Online, Idle or Invisible only while the per-account `owned-status` journal still owns DND. Respect pre-existing DND and observed manual changes. Manual changes that leave and return to DND between one-shot invocations are not detectable.
- Use the official Social SDK `SetOnlineStatus()` with OAuth2 PKCE and presence scope, keeping the refresh credential in the login Keychain. Discord public local RPC changes Rich Presence activity, not the account status. The installed helper uses the standalone signed arm64 release dylib and exits between changes. Packaging reduced disk size; the on-demand process removed idle SDK memory. [Social SDK API](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html) · [Local RPC surface](https://github.com/discord/discord-api-docs/blob/main/developers/topics/rpc.mdx)
- `local.focus-sync.sample` runs the built-in Focus Shortcut at load and when `WatchPaths` sees the DoNotDisturb `Assertions.json` or its parent DB directory change. The LaunchAgent can receive the path event even if direct file reads give `Operation not permitted`; the signed Shortcut reads Focus without Full Disk Access. Keep the sample event-driven. Running the Shortcut every five seconds posted live activities and drove `WidgetRenderer_Activities` into high CPU, swap, and memory growth. Restarting that renderer gives temporary relief; removing polling fixes the trigger.
- The sampler writes `on` or `off` atomically into `~/Library/Application Support/Focus Sync/focus-state`. `local.focus-sync.watch` reconciles on state change with a 15-second fallback, skips stale samples, and retries transient failures with a 20-second same-edge cooldown. The fallback checks recorded state and does not rerun the Shortcut or launch the SDK without an unapplied change.
- Inspect `launchctl print gui/$UID/local.focus-sync.sample` and `launchctl print gui/$UID/local.focus-sync.watch` and `~/Library/Application Support/Focus Sync/focus-sync.log`. A manual sample uses `shortcuts run 'Focus Sync Current Focus' --output-path <new-output-file> --output-type public.plain-text`; successful stdout may be empty. For recurring heat, inspect Shortcut assertions in `~/Library/Group Containers/group.com.apple.liveactivitiesd/liveactivitiesd.sql`; `Activities.request` carries base64 attributes containing the Shortcut title.
- Real Focus on/off and online/DND readback worked with event triggers, and DND persisted after the SDK disconnected. Reboot, logout/login and actual notification sound were untested.

[Prior install comparisons and dated verification](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/machines/swairM5/discord-focus-sync.md)
