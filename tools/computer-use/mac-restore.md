---
title: macOS Computer Use needs bundle IDs and refreshed UI state
scope: tools/computer-use
tool: macOS Computer Use
status: active
confidence: high
evidence: Direct localized-app lookup failure, stale-index error, refreshed state, and defaults/plist/GUI comparison.
created: 2026-08-18
updated: 2026-08-18
tags:
  - computer-use
  - macos
  - gui
  - verification
source_refs:
  - pilot-codex:events [42]-[46], [101]-[112]
  - pilot-codex:events [54]-[73], [121]-[138]
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Targeting

Use the macOS bundle identifier for localized applications. A localized display name failed to resolve while `com.apple.systempreferences` worked. A game lookup returned `Invalid app` and switching to the launcher ID then timed out. Listing an app or trying its ID does not establish screen access. Verify the actual target window or screenshot before claiming a visual check. This game case did not produce a successful screenshot. [Historical game probe](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/computer-use/desktop-app-negative-probe.md)

# State refresh

After navigation, scrolling, or any UI transition, request fresh application state before using element indexes. Reusing an old index produced `Apple event error -10005: cannotClickOffscreenElement`; refreshing the state and selecting the current element succeeded.

# Layered verification

Verify persisted settings through `defaults` or plist inspection and verify the System Settings GUI separately. A missing defaults key means no override exists; it does not prove the GUI value is false.

# macOS-specific restore details

Trackpad settings may need to be written and read in both observed preference domains:

- `com.apple.AppleMultitouchTrackpad`
- `com.apple.driver.AppleBluetoothMultitouch.trackpad`

Set and verify `ComputerName` and `LocalHostName` separately; they are distinct values.
