---
title: "Swair-M1 OBS Google Meet 錄影設定"
scope: machines/swair-m1
status: active
updated: 2026-09-29
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
