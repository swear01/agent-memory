---
title: ControlCenter brightness HUD repeats after an unmatched brightness-up event
scope: machine
machine: swairM5
tags: [macOS, ControlCenter, MenuBarAgent, brightness, HUD, keyboard]
status: active
updated: 2026-10-10
---

2026-10-10 在 swairM5（macOS 27.0.1，build 26A434）實際觀察右上角「顯示器」亮度 HUD 持續出現。視窗擁有者是系統 `MenuBarAgent`，提示由 `ControlCenter` 發送。

## 已驗證的觸發時間線

下列時間均為 Asia/Taipei，來自本機 unified log：

- 14:33:38.291：WindowServer 記錄 `[ Display:Power ] Broadcast: Will Wake`。
- 14:33:39.955：ControlCenter 的 `HotKeys` 類別記錄 `displayBrightnessUp (builtin, down)`。
- 14:33:39.956：ControlCenter 將亮度從 `0.368897` 調到 `0.437500`。
- 14:33:39.959：啟用 `DisplayBrightnessSystemBannerContent(value: 0.4375, range: 0.0...1.0, isEnabled: true)`。
- 14:33:40.459 起：首次約隔 0.5 秒，之後約每 0.083 秒（12 Hz）重複調亮與顯示 HUD；亮度升至 100% 後提示仍持續。
- 查詢 14:32 至 16:10 的 ControlCenter `HotKeys` 紀錄，只找到上述一次亮度鍵 down，沒有對應的 up，也沒有大量新的 brightness-down/up 通知。
- 15:57:10 至 16:09:10 的樣本有 8,606 次亮度 banner 更新，712 個完整秒各有 12 次；banner 的消失時間是 `custom(1.0)`，卻被連續刷新。
- 16:09:44 前後僅以 TERM 重啟 ControlCenter，新 PID 自動啟動；HUD 消失。後續超過一分鐘的 live log 沒有新的亮度 banner，螢幕仍正常上線。

## 結論與界線

直接觸發點是螢幕喚醒後的內建調亮熱鍵 down；ControlCenter 隨後維持未結束的長按重複狀態。沒有對應 up 的紀錄與重啟單一程序即可停止，支持放開狀態未被正常處理的判斷。這不是已證明的實體鍵故障，也不能進一步斷言是哪個元件遺失或攔截 key-up。

當時 BetterDisplay 4.3.6、Karabiner 和 `lid-display-switch` 均持續執行。Karabiner 配置僅把 Caps Lock 映射成切換輸入法，沒有亮度連發規則。虛擬螢幕背景程式每 300 秒執行狀態檢查；其 log 沒有對應的反覆顯示切換。重啟 ControlCenter 後這些程序未變動，HUD 即停止；尚無證據把根因歸給它們，也未完成隔離測試來完全排除它們。

未重現喚醒／亮度鍵操作，未取得異常當下原始 HID 按鍵序列；確切造成 key-up 未處理的元件仍待確認。不要把單純環境光亮度變化當成這次 HUD 連發的原因。

## 再發時的最小處理

先保留 ControlCenter 的 `HotKeys`、`display`、`system-banners` 和 MenuBarAgent 的 `systemBanners` 紀錄；需要時在重啟前取程序 sample，避免先清掉異常現場。用 `CGWindowListCopyWindowInfo` 確認 HUD 視窗擁有者。

若確認相同連發症狀，可執行 `killall -TERM ControlCenter`，等 launchd 自動重啟；選單列會短暫刷新。再查 live log 與可見視窗確認停止。這是本次已驗證的復原方法，不是已驗證的永久修正；不要為了隱藏 HUD 停用正常亮度控制或虛擬顯示器服務。
