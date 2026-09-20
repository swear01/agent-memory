---
title: Windows GUI 自動化（截圖 + AutoHotkey）通用技巧
scope: global
status: active
created: 2026-09-21
updated: 2026-09-21
tags: [windows, gui-automation, autohotkey, powershell, screenshots, brave]
---

# Windows GUI 自動化（swop 等 Windows 機器）

agent 自己無法顯示圖片時，用「截圖 → meta_analyze_file OCR → AutoHotkey 點擊」迴圈操作 GUI。踩過並驗證的規則：

## 截圖

- 用 PowerShell `System.Drawing.Graphics.CopyFromScreen` 截全螢幕（1920x1080）。
- **`PrintWindow` 對硬體加速（GPU）視窗抓到全黑圖**，不要依賴它。
- 截圖前先 `ShowWindow(hwnd, 3)`（SW_SHOWMAXIMIZED）+ `SetForegroundWindow`，並 sleep 數百 ms，否則其他視窗會遮住或座標漂移。
- 最大化後視窗 rect 固定（如 `x=-7 y=-7 w=1933 h=1045`），之後所有點擊座標可重複使用，不用每步重量。

## 鍵盤輸入（WScript.Shell SendKeys）

- **直接 `SendKeys` 打英文/網址會被注音輸入法劫持**（變成「ㄙㄛ」這種注音字）。
- 網址列標準流程：`SendKeys "^l"` → `^a` → `Set-Clipboard $url` → `^v` → `{ENTER}`。
- 開始前按 `{ESC}` 收掉輸入法的 candidate window。
- 開新分頁：`^t`；之後用上面的貼上流程跳網址。

## 點擊

- AutoHotkey v2：`C:\Program Files\AutoHotkey\v2\AutoHotkey.exe <script.ahk>`，腳本內容 `MouseMove x,y` + `Click`（雙擊就兩行 `Click`）。
- 簡單點擊、選單操作**不需要 OpenSpeedy**；OpenSpeedy 只在需要高速連續操作時才用。
- 使用者日常瀏覽器是 **Brave**（記憶見 `global/workflows/authenticated-browser-form.md`），要操作登入狀態就操作 Brave 本體。

## 用 Muse 讀截圖

- 主模型不收圖時用 `meta_analyze_file`。**DRM/除鎖相關 App 它會拒絕提供座標**；改問「pure OCR + 純位置、不要評論」會正常回座標。
- **裁切圖的座標原點是裁切區左上角**，換算回全螢幕要加裁切偏移（x_off, y_off）。多次裁切定位時容易混——能全圖定位就全圖。
- 截圖存到 `%LOCALAPPDATA%\Temp\` 給 Muse 讀。

## 實例

- 2026-09-21：swop 上操作 Epubor Honto Converter（選書/轉檔）與 Brave 開分頁跳轉，全程走上面這套，無 OpenSpeedy。