---
title: ASIOS Windows 存檔位置與卡住排查
scope: tools/asios
status: active
updated: 2026-09-07
---

# 已驗證的存檔位置

Auto Snakes in Outer Space（ASIOS）的 Windows 程序是 `asios.exe`。本次 Steam 安裝使用 LÖVE runtime，存檔位於 `%APPDATA%/Auto_Snakes_in_Outer_Space/save_v1.dat`，同目錄有 `steam_autocloud.vdf`。

排查沒有回應時，先把整個存檔目錄複製到獨立備份，再以 SHA-256 核對原檔與副本。備份只能保護已寫入磁碟的進度，無法保證保留尚在記憶體內的遊戲狀態；重啟前應查看存檔最後寫入時間。

# OpenSpeedy 排查的限制

2026-09-07 實際觀察：OpenSpeedy 倍速顯示 `1.000` 時，ASIOS 仍顯示 `Speed Enabled` 和 `Injection Yes`，因此 1 倍速不代表加速掛鉤已停用。

將 ASIOS 的獨立加速開關關閉後，介面確認 `Speed Disabled`，但遊戲持續沒有回應，存檔沒有更新。這次操作未恢復遊戲，不能据此認定 OpenSpeedy 是根因，亦不能將停用加速列為已驗證的修復方法。使用者選擇自行手動重啟；恢復與進度保留結果尚未確認。
