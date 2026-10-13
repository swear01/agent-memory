---
title: Google Photos Q70 並存原件清理、容量口徑與 B2 備份續接
scope: domains/media
status: verified-snapshot
updated: 2026-10-13
tags: [Google Photos, Q70, HEIC, HEVC, Takeout, Backblaze, B2, quota, recoverable-trash]
---

# Google Photos 原件清理與備份續接（2026-10-13）

## 已完成與授權範圍

- 本次使用者明確授權把「已確認有壓縮版並存」的原件移入可恢復垃圾桶，並接受此清單內非嚴重的日期、相簿及顯示尺寸差異；不要再僅以這些輕微差異阻擋已授權清理。這不是任意刪除全庫或永久刪除的授權。
- 10:58 Asia/Taipei 完成 409 個原件的 Google Photos 垃圾桶精確回讀：387 個影片、22 個照片項目，共 21,559,652,499 bytes（21.56 GB）；409 個對應壓縮版逐項確認仍在主圖庫。永久刪除為零，未清空垃圾桶。
- 22 個照片項目包括 15 個原先來源身分及 7 個同內容、不同雲端身分的其他舊副本。不能再用已在垃圾桶的歷史來源 ID 代替這 7 個目前副本。5 個遺漏的來源相簿關聯已先補到壓縮版並回讀。
- 20 個項目曾有日期或顯示尺寸等差異，依本次授權接受；原始中繼資料保存在凍結清單，不宣稱所有日期與原件完全一致。7 個額外照片副本的日期差異多為 0.094–0.637 秒；部分影片有更大的既有日期差異，不能泛化成未來一律忽略日期。
- 本次確認的並存原件清單已處理完；沒有逐張人工驗收全庫畫質，也沒有證明全庫零重複或所有照片都已轉檔。不同內容的同名版本仍保留。

## 根因與不應重跑的工作

- 初始「808 組」混入 supplemental JSON；更正為 404 組媒體名稱候選、403 個原件路徑。檔名相近不是重複證據，相簿也可能把同一項目重複匯出。
- 歷史上傳回條存在不等於原件已清理。這次 387 個影片原件中，384 個的新版未納入舊兩次集中 Takeout 回驗，另 3 個有舊 metadata hold；上傳、回驗、清原件流程曾未走完。2026-10-03 本機清理與雲端原件移入垃圾桶是不同操作。
- 這次稽核對 42,098 份歷史上傳回條及 121,723 個非垃圾桶匯出項目做交叉篩查，對選定 2,068 個檔案、35,878,570,185 bytes 完整回讀 CRC／SHA256，再核對目前來源與新版的雲端身分。這不是對全庫每個檔案都做 SHA256。
- 57 組同名候選是不同來源版本，不能按檔名刪除。3 組影片名稱變體及 3 個照片舊 URL 別名已用目前 Takeout 身分和完整雜湊釐清。
- 不要重跑已完成的歷史全量轉檔或退休來源下載佇列。原始專案的 SQLite、42,098 份上傳回條及舊 journal 尚在；新清理有獨立清單與回條，不改舊 frozen plan。未知寫入結果只回讀，不盲目重送。

## 容量口徑：檔案大小不等於計入配額的用量

- 排除本次已清理原件，按舊 Takeout 清單、大小及 CRC32 篩除相簿匯出副本，估算圖片 69,866,155,157 bytes（69.9 GB，約 4.74 萬份）、影片 60,366,106,412 bytes（60.4 GB），合計約 130.3 GB。這是內容大小估算，不是新匯出實測，也不是全庫 SHA256 去重或精確雲端項目數。
- Google 相簿頁面清理前顯示 100.34 GB，清理後顯示 97.6 GB，當時顯示差額約 2.74 GB。不要把 21.56 GB 原件總大小直接當成已釋放的配額。
- 保存的來源回傳資料中，194 個原件、18,577,120,942 bytes 的原始儲存 tuple 第二欄為 0；205 個原件、2,965,615,868 bytes 的該欄等於檔案大小。這支持大部分原件原本未計入配額的解釋，但欄位是未公開格式，不能當成正式帳務核算或判定每個項目的免計原因。
- Google 確有免計配額規則，例如 2021-06-01 前以 High quality／Express quality 備份的內容；不能只憑拍攝年份推定免費。批次刪除後配額最多可能需 48–72 小時更新，當時的顯示不是保證最終值。
- 官方依據：Google Photos Help「About your Google Photos activity & storage」answer/10100180；Google Drive Help「Manage your storage」answer/6374270。需要當前用量時重新讀取頁面，勿把上述快照當成永久現況。

## Backblaze B2 尚未整體完成

- 備份目標維持全 Drive 排除 Share，加上清理後 Google Photos 的非垃圾桶完整 Takeout；既有 Git 目錄保留。普通影片沿用最高 1080p、不放大、硬體 HEVC Q60；CVSD 若候選更大則依已授權決定保留較小原片，不擅降品質。
- 已有回條證明新 Drive.tar.zst 為 45,388,302,310 bytes、42,522 檔／6,625 目錄，完成 B2 全量 SHA256 回讀及還原檔案 SHA256 驗證。SHA256 為 `8e29b230cba2c607f88502483c9c194f08dba7bcbd29820a1b09f56fcdec8e50`。DriveExtras、NativeForms、DriveSupplement 與 Maps 另有各自回讀證據，不能由較早進度檔的 pending maps 推回未完成。
- 舊 3 月 B2 備份 65 個物件、535,175,554,940 bytes，在本次稽核回讀時身分／大小未變。完整新 Photos 備份、回讀及還原未完成前，不替換或刪除舊備份，不動無關 GitHub 備份。
- 依最新本機回條，清理後又於 2026-10-13 11:15 Asia/Taipei 建立 Google Photos 新匯出：34 個相簿選項、僅排除 Trash、Add to Drive／一次／ZIP／50 GB，身分驗證完成。當時 Google 正在建立，備份 worker 未啟動。這是回條快照，本次記憶更新沒有另查即時完成狀態。
- 後續優先核對 11:15 最新作業，不誤用 09:33／09:37 舊作業，也不要自行再建立重複匯出。按最新作業凍結來源 ZIP 身分、確認全部中央目錄無 Trash，再執行受保護的備份腳本；每卷核對 Drive 大小／MD5、ZIP CRC、B2 全檔 SHA256 回讀及每個還原 member 的 SHA256。

## 可續接證據位置

本輪工作區為 `<workspace-root>/work/backblaze-refresh-20261012/`，以下相對於該目錄；原始遷移專案為 `<home>/Documents/Codex/2026-09-06/google-drive-api-thread-01a0729b-38d4`。只保存相對路徑與統計；不把登入憑證、媒體身分或私人 URL 複製到共享記憶。

- `photos-coexisting-trash-20261013/`：`plan.json`、`result.json`、`completion-check.json`、`trash-readback.json`、`replacement-survival.json`、`album-readback.json`、`retained-media-size-estimate.json`、`quota-after-cleanup.json`。409 個來源的獨立逐件 journal 已標為 trashed_verified；本次暫存登入檔已移除。
- `takeout-inspection/full-library-audit-20261013/`：`audit-result.json`、`completion-check.json`、`global-original-content-audit.json`、完整 member hashes、B2 保留回讀。這個 read-only audit Goal 已完成，不能因此把仍等 Photos 的整體備份標成完成。
- `photos-no-trash-backup/expected-export.json`、`progress.json`、`self-check.json` 及根目錄 `goal-checkpoint.json`：最新 11:15 匯出、worker 未啟動與下一步；不要只讀較早 Takeout 狀態快照。
- `drive-current/b2-receipt.json`、`owned-extras/b2-receipt.json`、`maps-b2-receipt-20261013.json`、`native-forms-b2-receipt.json`、`supplement-b2-receipt.json`：分項備份驗證；整體 Photos 備份仍待完成。
