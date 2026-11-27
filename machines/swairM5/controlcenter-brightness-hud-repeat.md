---
title: ControlCenter brightness HUD repeats after an unmatched brightness-up event
scope: machine
machine: swairM5
tags: [macOS, ControlCenter, MenuBarAgent, brightness, HUD, keyboard]
status: active
updated: 2026-10-10
---


螢幕喚醒後，亮度 HUD 即使到 100% 仍反覆刷新。用 `CGWindowListCopyWindowInfo` 確認視窗擁有者 `MenuBarAgent`，保留 ControlCenter 的 `HotKeys`、`display`、`system-banners` 和 MenuBarAgent 的 `systemBanners`。需要時在重啟前取程序 sample。

已記錄的症狀是單次 `displayBrightnessUp (builtin, down)`，無對應 up，約 0.5 秒後進入約 12 Hz 重複。這支持 ControlCenter 維持長按狀態。確切的 key-up 遺失位置、硬體故障、BetterDisplay／Karabiner／虛擬顯示器因果均未驗證。

同症狀下可 `killall -TERM ControlCenter`，等待 launchd 自動重啟及選單列刷新。檢查 live log 已停止更新 banner、HUD 消失且螢幕保持上線。這是已驗證的單次復原。保留正常亮度控制及虛擬顯示器服務。

[原始驗證紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/swairM5/controlcenter-brightness-hud-repeat.md)
