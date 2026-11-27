---
title: "Apple Silicon (M1/M5) OBS 螢幕錄影最佳化與硬體編碼避坑設定"
scope: tools/obs
status: active
updated: 2026-11-28
---

## Apple Silicon 錄影掉格或畫面變形

- **OBS 日誌有 `P010 texture support not available`、encoding lag 或 rendering lag。** 若錄 SDR，先用已驗證的 NV12、Rec.709、Limited、Apple VT HEVC Hardware、30 fps。由 4K60 降到 1920×1080 或 1920×1200，依素材選比例，畫布和輸出用同一比例。看實際掉幀統計與長度代表性的錄樣，別以「已選硬體編碼」判定通過。
- **錄好才發現檔案無法開啟。** 用 MKV 錄影，正常停止後再自動 remux MP4。需單檔時把自動分割關掉。原測試用 CRF 55–60、keyframe 2 秒、CoreAudio AAC 160 kbps。該版本 CRF 不適用時的替代是 CBR 3500–5000 kbps，重驗文字可讀性與音軌。
- **已錄成多段。** 先查 segments 的 codec、解析度與音軌一致。把各段依序列入 concat 清單，再用 `ffmpeg -f concat -safe 0 -i <list.txt> -c copy <merged.mkv>`。回讀合併長度、音軌與段落接點。

## M1 雙螢幕 Google Meet 錄影

- **`Failed to create source` 或重開 Brave 後黑畫面。** 在 OBS 32 的既有設定，用 ScreenCaptureKit 視窗擷取（`screen_capture`、`type=1`、`com.brave.Browser`）取代 legacy `display_capture`。它記 window ID，換 Meet 視窗或 Brave 重開後要重選。應用程式擷取備援（`type=2`）只跟綁定螢幕，音軌保持靜音以免重複。
- 用 1920×1200、30 fps。擷取來源音訊可錄會議，自己的聲音另看 Mic/Aux 是否開啟。正式錄影前短錄並核對輸出的解析度、非黑畫面與音軌峰值。
- **改了 profile 卻被覆寫。** 先完全退出 OBS 再改 `basic/profiles/<profile>/basic.ini`、`recordEncoder.json` 或場景 JSON。遠端快捷鍵依賴 Accessibility。若異常終止，先保留 MKV，確認檔案能讀再處理封裝。
- **程序還在，但整段沒錄進檔。** 暫停時日誌有 `output … paused`、`unpaused`，檔案不再長。焦點留在 OBS 時空白鍵可能觸發 Controls。用獨立熱鍵或連錄後裁切，現場看狀態圖示，收尾核對錄影長度。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/tools/obs/apple-silicon-hardware-recording-settings.md)
