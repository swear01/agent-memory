---
title: macOS 桌布、鎖定畫面與螢幕保護程式（純黑）
scope: tools/macos
status: active
updated: 2026-10-29
tags:
  - macos
  - wallpaper
  - screensaver
  - lockscreen
---

# macOS 桌布、鎖定畫面與螢幕保護程式（純黑）

## 教訓（2026-10-29，M5）
- 只改 Desktop 不夠：待機和鎖定畫面還是內建空拍 Golden State（金門大橋），我兩次回報做好了，其實都沒有生效。
- 設定檔：`~/Library/Application Support/com.apple.wallpaper/Store/Index.plist`（binary plist，要用 Python 的 `plistlib` 讀寫）。
- 真正生效的是頂層的 `AllSpacesAndDisplays`，裡面 `Type: idle`、`Idle.Content.Choices[0].Provider` 是 `default`，`default` 代表內建空拍。只改 SystemDefault、Spaces、Displays 底下的 Idle 都沒用。
- 修法：把 `AllSpacesAndDisplays.Idle.Content` 換成跟 Desktop 一樣的 image choice（`com.apple.wallpaper.choice.image`，指向 `~/Pictures/pure-black.png 的 file URL`），改之前先備份，改完 `killall WallpaperAgent`。
- 回報前一定要驗證：`open -a ScreenSaverEngine`，等 6 秒後 `screencapture -x`，看截圖是不是全黑，然後 `killall ScreenSaverEngine`。
- 改完再全部列一次所有 Provider，只要還有任何一個 `default`，就代表沒改完，不能忽略。
- 手動做法：系統設定 → 螢幕保護程式，選一個非空拍的；系統設定 → 桌布，用「加入照片」選 `pure-black.png`。
