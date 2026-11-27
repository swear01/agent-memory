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

## 2026-10-12 最後一批瘦身結案

68 支影片已按 1080p 上限修正：36 支用 1080p 原件重轉，28 支原生 720p 與 4 支 480p 無升頻。原 Drive ID／名稱／父目錄及來源保留，全部雲端比對通過。PDF／PPTX、FLAC、STEP、ARK 與 LDA 詳細驗收在原回條；教育資料集見 [學校整理收尾](google-drive-school-cleanup.md#2026-10-12-資料集封存與最終收手)。這是歷史結案，重查使用量或來源位置以 live 為準。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/domains/storage/google-drive-cleanup-202610.md)
