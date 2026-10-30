---
title: DVLab Windows laptop LAPTOP-U5DAT43H: auto logon and 26H2
scope: machines/laptop-u5dat43h
machine: laptop-u5dat43h
status: active
updated: 2026-10-30
---

# LAPTOP-U5DAT43H (new DVLab Windows laptop, ASUS, i7-11370H)

- Windows 11 Home 25H2 (registry ProductName still says "Windows 10 Home" — normal quirk; trust CurrentBuild 26200 / DisplayVersion).
- Account `dvlab` = Microsoft account the lab Microsoft account; shell sessions are not elevated — admin changes need `Start-Process -Verb RunAs` (user clicks UAC).
- Auto logon (2026-10-30): set HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device DevicePasswordLessBuildVersion=0 (turns off "only allow Windows Hello sign-in", which hides netplwiz checkbox), then user unchecks "must enter user name and password" in netplwiz and types the MS account password → AutoAdminLogon=1, password in LSA secret (no DefaultPassword plaintext).
- 26H2 upgrade (2026-10-30): offered via WU (enablement package), GE26H2 indicators show no safeguard hold; was blocked by a pending reboot (`...WindowsUpdate\Auto Update\RebootRequired`). Check offers via COM `Microsoft.Update.Session` searcher "IsInstalled=0 and IsHidden=0".
