---
title: "Swair-M1 OBS Google Meet 錄影設定"
scope: machines/swair-m1
status: active
updated: 2026-09-30
---

# Swair-M1 OBS Google Meet 錄影設定

## Machine

- Host label：`Swair-M1.local`
- Chip：基礎款 Apple M1（單組 AVE）
- OBS：32.2.2
- Browser for Meet：Brave (`com.brave.Browser`)

## Durable rule

細節與避坑寫在 `tools/obs/apple-silicon-hardware-recording-settings.md` 的「M1 雙螢幕 Google Meet 錄影」段。本機目前生效的重點：

1. 畫布／輸出 `1920×1200`、30 FPS、NV12、Rec.709 Partial。
2. 編碼：Apple VT HEVC Hardware、CRF 60、keyint 2s、MKV + AutoRemux、不分割。
3. 主要來源：ScreenCaptureKit 視窗擷取 Brave 的 Meet 視窗（含會議音訊）；麥克風靜音。
4. 不要用 legacy `display_capture`。Brave 重開後要在 OBS 重新選視窗。
5. 改設定前必須先退出 OBS；快捷鍵 `U`/`I` 開始／停止錄影。

## Incident: 2026-09-29 暫停缺口（15:07–16:12）

- 症狀：課中少錄約 64 分鐘；使用者感覺「暫停鈕怪怪的」。
- 日誌：`Library/Application Support/obs-studio/logs/2026-09-29 14-15-07.txt`
  - `15:07:30 output adv_file_output paused`（**沒有** `due to hotkey`）
  - `16:12:00 unpaused`，隨即切 Studio Mode，`16:12:05` 又 paused → `16:20:51` unpaused
  - 同一檔 `14-21-34.mkv`：drawn frames≈272862、output frames≈140973（30fps）→ 暫停約 73 分鐘未寫入
- 熱鍵現況：只綁 `U` 開始、`I` 停止；**未綁 Pause/Unpause** → 暫停來自 Controls 小按鈕（或 OBS 前景時空白鍵點選聚焦按鈕）。
- 根因：暫停功能正常；長時間暫停且 OBS 在背景時狀態不易察覺，造成「以為還在錄」。
- 實務：上課不要用暫停（一路錄再剪）；若要用，設專用暫停熱鍵（勿用空白鍵），並確認狀態列是紅點而非暫停條。
