---
title: CyberpunK Google Drive 封存整理完成狀態
scope: project
project: cyberpunk-drive-cleanup
tags: [google-drive, heif, fileprovider, archive, sevenzip]
status: active
created: 2026-09-03
updated: 2026-09-03
---

# CyberpunK Google Drive 封存整理完成狀態

## 完成結果

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
