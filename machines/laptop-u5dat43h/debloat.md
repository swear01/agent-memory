---
title: LAPTOP-U5DAT43H tiny11-style debloat (2026-10-30)
date: 2026-10-30
tags: [windows, laptop-u5dat43h, debloat, asus]
---
# LAPTOP-U5DAT43H debloat

The goal is a laptop as clean as tiny11, with Windows Update, Store, Security, Edge and the NVIDIA graphics driver still working.
The local exec shell is not elevated, so each cleanup pass was a script started with `Start-Process -Verb RunAs`, and the user clicked UAC. Transcripts are `C:\cleanup*-20261030.log`.

## Removed
- Office LTSC 2021, OneDrive, WSL plus VirtualMachinePlatform, and the Ubuntu apps.
- Copilot, Bing, Xbox and Gaming Services, Teams, Clipchamp, Outlook, Messaging, Whiteboard, Paint 3D and Wallet.
- Arc, Arduino IDE, Kyocera Print Center, PicPick, Vim, WinRAR and the Google PWA entries.
- McAfee, verified gone: no services, appx, folders or uninstall entries.
- SIMetrix, the WibuKey dongle driver (via pnputil), Visual Studio Installer, and the GeForce Experience, PhysX and FrameView packages.
- ProArt Creator Hub: its MSI returns 1603, so the services were deleted and the folder and uninstall key removed by hand.
- The leftover accounts Client and defaultuser100001.
- Everything in the user folders.

## Kept on purpose
- Grok Bot, Edge, the NVIDIA driver and control panel, Update Health Tools, and KB5001716.
- Store, SecHealthUI, Calculator, Notepad, Paint, Photos, Terminal, Snipping Tool, Camera, the codecs and the language packs.
- The driver companion apps: Realtek, Intel graphics, Thunderbolt and DTS.
- ASUSOptimization. The user wants the Fn hotkeys working, and they depend on it. The other MyASUS services (AsusAppService, SoftwareManager, Switch, SystemAnalysis, SystemDiagnosis) are disabled.

## Start and taskbar
- The user wanted every pin removed.
- Start pins were cleared by replacing start2.bin with the empty one from Win11Debloat's `Assets/Start/start2.bin`. A backup of the old file is in the user profile.
- The OEM `LayoutModification.xml` (it held the ASUS, McAfee and Xbox tiles) was emptied.
- Taskbar pins were cleared by deleting the User Pinned\TaskBar shortcuts and the Taskband Favorites values, then restarting Explorer.

## Gotchas
- A name regex containing `Arc|Access` also deleted the built-in Start menu folders Accessibility and Accessories. Their main shortcuts were recreated by hand. Anchor name patterns in the future.
- Two OneDrive shell DLLs in Program Files\Microsoft OneDrive stay locked by Explorer; delete them after a reboot.
- A reboot is still pending for the WSL/VMP feature changes and the WibuKey driver removal.
