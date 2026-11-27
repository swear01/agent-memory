---
title: CyberpunK Drive 壓縮：保留檔案身分與分批驗證
scope: project
project: cyberpunk-drive-cleanup
tags: [google-drive, heif, fileprovider, archive, sevenzip]
status: active
created: 2026-09-03
updated: 2026-10-11
---

- **Drive 容量或待處理數與上次回條不同。** 重新列遞迴當前目標，分開量本地 FileProvider pending 和雲端檔案狀態。九月與十月快照不可直接相加。Drive Trash 仍可能占容量，先確認而不要依舊快照永久刪除。
- **壓縮重建後怕丟原 ID、CAD 內容或簡報畫面。** 照片留在原 Drive、用原尺寸 HEIF，GIF 保留。影音與 PPTX 逐檔核對來源 ID、雲端大小及雜湊，重新渲染簡報抽查畫面；確認其餘二進位物件沒有變動。RAR 改 7z 時，以 `7zz t` 加解壓後 `diff -qr` 驗內容。照片與非影音整理不能證明 YouTube 上傳。

[原始紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/cyberpunk-drive-cleanup/status.md)
