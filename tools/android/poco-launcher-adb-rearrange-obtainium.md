---
title: "用 ADB 自動重排 POCO 桌面並把 App 加進 Obtainium"
scope: tools/android
status: active
updated: 2026-11-21
---

# 用 ADB 自動重排 POCO 桌面並把 App 加進 Obtainium

2026-11-21 在 POCO F7（Android 16、POCO launcher `com.mi.android.globallauncher`、未 root）上實測可行的做法。

## 連線

- USB ADB 常斷，改成 Wi-Fi ADB：`adb tcpip 5555` 然後 `adb connect <phone-ip>:5555`。第一次 tcpip 可能無聲失敗，重跑一次。
- 出現 offline 或 `No route to host` 時，先 `adb kill-server` 再 connect，通常就能恢復。
- 截圖直接用 `adb exec-out screencap -p`，經過 retry 包裝的輸出會壞掉。

## 讀畫面

- `uiautomator dump /sdcard/ui.xml`，再用 `adb exec-out cat /sdcard/ui.xml` 讀出來，解析 TextView 的 text 和 bounds。
- 1280 寬的螢幕上桌面格子是 4×7，格子中心 x=188+(c-1)*301，y=328+(r-1)*302。
- 拖曳時整個 workspace 會以 (640,2271) 為中心縮成 0.86 倍，放下的目標座標要照縮放後的位置算。

## 拖曳圖示

- 先在 launcher 設定（HomeSettingsActivity）關掉「Lock Home screen layout」，不然任何拖曳都沒反應。做完記得再鎖回去。
- `input draganddrop` 在這個 launcher 上沒用。要用 `input touchscreen motionevent`：先 DOWN，sleep 0.9 秒進入拖曳，接著用多段 MOVE 平滑移動，到目標上停一下再 UP。如果直接跳到目標位置，圖示會彈回原處。
- 停在資料夾上約 1.6 秒，資料夾會打開。要把手指移進打開的面板裡（例如 (640,1450)）再放開；第一排的資料夾特別要注意，不然會放在標題區而失敗。
- 從資料夾裡拖出來時，先移到 (640,2550) 等 2 秒讓資料夾關掉。
- 換頁的方法是停在螢幕邊緣（x=12 或 1268）。停 0.8 秒剛好翻一頁，停 1.4 秒會多翻一頁。
- 改資料夾名稱：長按資料夾，選 Edit folder，點標題欄，按 `keycombination 113 29`（Ctrl+A）再按 DEL，然後 `input text`。`input text` 打不了中文，中文名稱要使用者手動改。

## Obtainium

- 用 deep link `obtainium://add/<repo-url>` 開啟新增頁面（`am start -a android.intent.action.VIEW -d ...`）。
- 「Include prereleases」預設是開的，會選到 dev 版本（例如 MicroG 7.3.2-dev.1）。要先關掉這個選項，再挑正式版的 APK（MicroG RE 要選 icon 版本）。

## 判斷 App 來源

- `pm list packages -3 -i` 列出安裝來源，再看 dumpsys 裡的 initiatingPackageName。`com.miui.huanji` 是小米換機（Mi Mover）搬過來的；這類 App 原本可能是從 Play 裝的，不要直接當成側載。
