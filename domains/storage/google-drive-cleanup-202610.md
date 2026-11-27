---
title: Google Drive 2026 年 10 月整理總結與保留邊界
scope: domain
status: verified-with-limits
updated: 2026-10-12
tags: [google-drive, cleanup, anime, mega, goodnotes, lda, sparse, motion-photo]
---

# Drive 整理：保留原結構、回讀和配額口徑

- 壓縮後原位更新同一 Drive ID、名稱與父目錄，不在旁邊留「新版」。使用原來源，不對較低解析度的重壓檔做上採樣。移除採可還原垃圾桶，2026-10-12 使用者已要求結束整輪，勿重開歷史候選或擴及其他目錄。沒有清空垃圾桶的授權。
- Git 保留展開的 `.git` 與可直接使用的原專案結構。Music-to-Lyrics 獨立 `.git.7z` 已撤回，原 ID/路徑/MD5 回讀一致；不可把實驗封存或短暫節省當現行結果。去重前查檔案引用、資料集分區和同名異內容；封存後完整解壓、路徑與逐檔 hash、再完整雲端讀回後才清已授權來源。
- 一般影片以 1080p 上限、不放大、硬體 `hevc_videotoolbox` Q60／B2、停用軟體 fallback，保留原幀率、音訊封包與時間戳。先前 720p 已更正；重做要從高解析度原件。快速 seek 因 B-frame 可能對到不同畫面，比對同解碼幀索引，另驗完整解碼、source/target 音訊與 readback。Readback 遇 `RemoteDisconnected` 可用 HTTP/2 Range，逐段核 `Content-Range` 和全部 MD5／SHA。
- PDF／PPTX 用 [照片品質標準](../media/photo-compression-standard.md)，保留文字、向量、非圖片部件、透明度及編輯能力。影片剪輯 archive 保留專案檔不代表 OpenShot／Resolve 可直接開或重連；素材容量與專案本體分開。WAV 少數確定可換才轉 FLAC，未驗消費端的 Resolve、pygame、固定 `out.wav` 一律留原格式。不應把全量渲染或格式支援外推為原生動畫、內嵌影片、HDR、遊戲開啟、GoodNotes 匯入皆已實測。
- Drive 舊父目錄已進垃圾桶時，子檔仍可有獨立 `ownedByMe`／`trashed` 狀態。全量快照加 changes token 續接，批次 API 外層 200 還要逐項查 inner status 和操作後 metadata。Blob size 和配額 `quotaBytesUsed`／`usageInDrive - usageInDriveTrash` 分開，配額可能延遲更新。
- 歷史個案：LDA 用 `lda_storage.save_model/load_model` 保留原檔名，還原 ndarray、RNG、float64 位元。NPZ 壓縮若排序 COO 資料，重建設 `sorted=False`；驗 full-row fill、截斷／損毀 footer 和舊新 scikit-learn inference。Minecraft 只重建有版本和原下載 checksum 的 JAR，未知 JAR、歷史世界與設定留存。Anime 出庫有使用者確認 MEGA 上傳，未有獨立 MEGA 讀回；一份未解密舊包的風險接受只適用當批，不是後續刪除政策。

## 解壓、文件結構與 JPEG Motion Photo

- 去重查大小、完整 hash、程式引用、資料集分區。ZIP 與解壓樹比對完整路徑／目錄／CRC／逐檔內容；資料封存要驗還原與雲端讀回才清當批授權的來源。文字封存保留編碼和引用；PDB 被移除便失去原 binary 的符號資訊，C source 存在不證明可重編。
- OOXML 保留 XML default namespace，ElementTree 寫成 `ns0:Types`／`ns0:Relationships` 曾使 LibreOffice 載入失敗。保留非影像部分 bytes，驗字型、全頁渲染與回下載。原生 PowerPoint 動畫、PDF 內嵌播放器與一般 1080p HEVC 檔案各需驗。
- JPEG Motion Photo 要解析實體 MP4、HDR gainmap 和 MPF。若保留尾部元件，重算 MPF offset，依 ExifTool、byte comparison 和 decode 驗證；HDR gainmap 可見不等於 bytes 相等或手機實測成功。是否可以移除動態部分看該批授權。

[校內講義、原位封存與素材復原收據](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/domains/storage/google-drive-school-cleanup.md)

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/domains/storage/google-drive-cleanup-202610.md)
