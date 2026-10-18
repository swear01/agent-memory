---
title: "PiliPlus 台灣播放卡頓：緩衝上限與標準設定檔位置"
scope: tools/piliplus
status: active
updated: 2026-10-19
---

# PiliPlus 台灣播放卡頓：緩衝上限與標準設定檔位置

使用者的 PiliPlus 完整設定檔以 Google Drive `我的雲端硬碟/document/Setting Backup/app setting/piliplus-minimal-complete.json` 為唯一主檔，從 `~/Library/CloudStorage/GoogleDrive-*/` 尋找掛載。重建腳本與詳細偏好記錄在 Codex memories（`~/.codex/memories/extensions/ad_hoc/notes/*piliplus*`）。匯入會取代整個 `setting`/`video`，修改前先讀最新主檔、保留其他設定。

## 卡頓根因（官方原始碼 2.1.6.x）

- `Pref.initBuffer` 實際緩衝 = min(`bufferSize` MB, `bufferSec` × 倍速)；預設 4MB / 16s。1080p 時 4MB 只約 10 秒內容，2 倍速只剩約 5 秒實際時間。
- 2026-10-19 從台灣 Mac 單連線測速：熱門影片 aliov 約 8 MB/s、cosov 約 9 MB/s；冷門影片所有節點約 0.2–1 MB/s（大陸 ali 偶爾 3 MB/s 但會失敗），hwov、hk_bcache、08c 連線失敗，akamai 403。冷門影片回源慢，換 CDN 效果有限。
- 預設 `preferCodecs` 是 `[AVC, AV1]`，AVC 位元率最高。

## 2026-10-19 套用

`bufferSize: 64.0`、`bufferSec: 60.0`、`preferCodecs: ["HEVC","AVC"]`（未放 AV1，因手機硬解支援未知）。CDN 保持 `aliov`。主檔 SHA-256 `b57178cf80d895e91e575496154c0254f807662dff08bb8932d60fc2721c8c2a`。只驗證檔案；手機匯入與實際播放未驗證。根治需上游 PR #3223「线程撕裂者」多線程分片下載（2026-10-19 尚未合併）。
