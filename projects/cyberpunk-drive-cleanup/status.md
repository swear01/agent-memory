---
title: CyberpunK Google Drive 封存整理完成狀態
scope: project
project: cyberpunk-drive-cleanup
tags: [google-drive, heif, fileprovider, archive, sevenzip]
status: active
created: 2026-09-03
updated: 2026-10-11
---

# CyberpunK Google Drive 封存整理完成狀態

## 2026-09-03 歷史完成結果

- 照片不傳 Google 相簿；在原 Google Drive 資料夾直接以原尺寸、品質 15 的 HEIF 取代 JPG／JPEG／PNG。
- 最終圖片共 2,389 張：2,387 HEIC、2 GIF，合計 959,344,589 bytes。所有 HEIF 可讀，Google Drive FileProvider 待上傳為 0。
- 照片由 10,067,211,933 bytes 降至 959,344,589 bytes，節省 9,107,867,344 bytes（約 8.48 GiB，90.47%）。
- 非照片影片內容由 20,246,371,811 bytes 降至 3,361,107,842 bytes。已清理舊軟體、簡報歷史版本、程式安裝／封存包、明確副本與設計組 Garbage；正式簡報、程式、文件及主要 CAD 資源保留。
- ANSYS／SpaceClaim 工程完整保留，但封裝由 RAR 改為 7z：57 files、168,906,613 uncompressed bytes、53,110,637 archive bytes。驗證包含 `7zz t` 以及重新解壓後 `diff -qr` 逐檔相同比較。
- Google Drive Desktop 最終為已更新、無同步錯誤。刪除項目只移至垃圾桶，未永久清空；垃圾桶仍可能計入帳戶容量。

## 後續邊界

- 這個狀態只涵蓋照片與非影音檔案整理，不代表 YouTube 影片上傳已重新驗證。
- 未來再次清理前先重新量測 Drive 現況；不要根據這份快照直接永久刪除垃圾桶。
- Drive Desktop 的介面批次數不是完整待處理總數。大型作業要同時確認 FileProvider 檔案狀態與 Drive 顯示已更新。

## 2026-10-08 至 09 續作完成快照

本輪回條以 `<user-documents>/Codex/2026-10-08/new-chat/outputs/` 為準；上方九月內容與容量不能當作十月最新清單，也不能與十月各批節省量重複相加。

- 影音壓縮第一批 148 個目標中 147 個縮小後原 ID 更新，已驗證節省 45,319,173,748 bytes；後續另完成 472 支影片及 4 份 PPTX 的原位替換。後批 168 張投影片渲染像素相同，476 個替換檔案雲端大小／雜湊通過，其他 9,808 個二進位檔案雜湊不變。
- 最後移入垃圾桶的是 205 個確認重複檔與 1 個空檔；另修復處理節省 24,627,305 bytes。最終活躍資料夾 10,111 檔、15,421,731,174 bytes，非永久刪除。
- 587 個受保護重複群、5,025 個檔案保持原狀；不能將全球相同雜湊自動當成可刪副本。原生 Google 文件內容、PowerPoint 播放及外部剪輯專案引用未驗證；不宣稱所有應用功能已通過。
- 單一本機損壞原始 MTS 備份也已雜湊核對後進 macOS 垃圾桶，修復後雲端檔保留。回條：`cyberpunk-compression-replacement-20261008.json`、`cyberpunk-phase2-final-20261008.json`、`cyberpunk-cleanup-final-20261009.json`、`cyberpunk-local-backup-trashed-20261009.json`。
