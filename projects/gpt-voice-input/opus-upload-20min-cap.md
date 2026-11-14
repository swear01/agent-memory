---
title: "錄音上傳改 Opus 32 kbps、上限 20 分鐘，長錄音截斷未驗證"
scope: projects/gpt-voice-input
status: active
updated: 2026-11-14
---

# 錄音上傳改 Opus 32 kbps、上限 20 分鐘，長錄音截斷未驗證

原本每段錄音寫死 120 秒上限；48 kHz WAV 約 4 分 20 秒就會碰到 OpenAI 的 25 MB 上傳限制。v1.0.16（PR #19）改成 MediaCodec Opus 32 kbps + MediaMuxer Ogg，邊錄邊壓，20 分鐘約 5 MB；上限改 20 分鐘，minSdk 26 → 29。

使用者的決策與理由：只有自己用，不考慮舊手機；流量越低越好，選 Opus 32 kbps 而非無損或 48 kbps；不要分段上傳，設 20 分鐘上限就好。

未驗證（截至 2026-11-14）：
- 真機上的 Opus 編碼與 Ogg 封裝輸出；單元測試只用假的 PcmSink。
- gpt-transcribe 對 10～20 分鐘音檔的表現。OpenAI 沒寫長度上限，但社群回報 gpt-4o-transcribe 超過 1500 秒會報錯，且約 8～9 分鐘後的文字會被截掉。使用者接受這個風險；若真機實測確認會截，再改成分段上傳。
- Opus 和 WAV 的辨識品質沒有做 A/B 比較。
