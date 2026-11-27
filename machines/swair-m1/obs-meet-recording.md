---
title: Swair-M1 的 OBS Meet 錄影入口與熱鍵
scope: machines/swair-m1
status: active
updated: 2026-11-28
---

- `Swair-M1.local` 在 OBS 32.2.2 用 Brave 的 ScreenCaptureKit 視窗擷取、會議音訊開啟、Mic/Aux 靜音。此機驗證的是 1920×1200、30 fps、Apple VT HEVC Hardware、CRF 60、MKV 自動 remux。
- `U` 開始、`I` 停止。曾沒綁 Pause 熱鍵仍在 Controls 暫停，程序繼續跑但檔案漏錄。錄課建議連錄後裁；使用暫停時看狀態圖示，完成後核對實際長度。
- 黑畫面、掉幀與換視窗重選見 [OBS 錄影排錯](../../tools/obs/apple-silicon-hardware-recording-settings.md#m1-雙螢幕-google-meet-錄影)。

[當次錄影缺口與日誌](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/machines/swair-m1/obs-meet-recording.md)
