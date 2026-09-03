---
title: HEIC 批次處理要依內容辨識格式
scope: domains/media
tags: [HEIC, HEIF, JPEG, ImageMagick]
status: active
created: 2026-09-03
updated: 2026-09-03
---

# HEIC 批次處理要依內容辨識格式

不要只依副檔名決定處理方式。實際遇過一批 `.HEIC` 檔，其內容是 JPEG；`file -b --mime-type` 回報 `image/jpeg`，而 `heif-info` 會以缺少 `ftyp` box 拒絕讀取。

安全的最小流程：

1. `heif-info` 能讀取來源時，視為真正 HEIF，直接複製以避免再次有損壓縮。
2. 若讀取失敗，再用 `file -b --mime-type` 判斷；內容是 JPEG 時才轉碼成 HEIF。
3. 一律先寫暫存檔，以 `heif-info` 驗證成功後再原子改名；不要覆寫來源。

2026-09-03 的實際批次中，11 個誤標 `.HEIC` 檔依此修正後全部通過驗證。
