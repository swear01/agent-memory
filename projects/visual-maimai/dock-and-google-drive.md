---
title: Visual Maimai Wineskin Dock and Google Drive contract
scope: projects/visual-maimai
project: Visual Maimai
machine: macOS
status: active
confidence: high
evidence: Full stop/restart and functional verification on 2026-08-20
created: 2026-08-20
updated: 2026-08-20
tags:
  - visual-maimai
  - macos
  - wineskin
  - porting-kit
  - wine
  - dock
  - google-drive
---

# Verified result

The Visual Maimai Wineskin wrapper initially lacked the outer `NSBGOnly=1`
metadata present in the compatible Porting Kit master wrapper. Both wrappers
use the same launcher binary. The missing outer metadata caused the launcher
and the Wine GUI application to appear as separate Dock/application entries.

After backing up the plist and adding the exact compatible key:

```xml
<key>NSBGOnly</key>
<string>1</string>
```

then fully closing and reopening the app on 2026-08-20:

- the launcher was absent from the visible application list;
- only the foreground Wine GUI application remained as the Dock entry;
- the Visual Maimai window rendered normally;
- a Wine listing of the mounted Google Drive CloudStorage directory returned
  exit code 0 and listed 10 directories.

# Decision

Keep `NSBGOnly=1` for the current macOS wrapper. It is a legacy key, but the
compatible Porting Kit pattern works on this system. Do not proactively replace
it with `LSBackgroundOnly` or patch/replace `winemac.so`; revisit a modern Launch
Services method only if a future macOS version stops honoring this key. Do not
substitute a normal Wine driver for the CrossOver/Konyak/DXMT runtime because of
ABI and graphics compatibility risk.

The Google Drive location is under macOS CloudStorage rather than a legacy
mounted-drive location. With the existing launcher Full Disk Access and this
wrapper configuration, a separate Full Disk Access grant for `wineserver` was
not required by the successful Wine directory test.
