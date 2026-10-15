---
title: Google Photos Q70 原件清理、完整 B2 備份與 Oracle 暫存收尾
scope: domains/media
status: verified-closeout
updated: 2026-10-15
tags: [Google Photos, Q70, HEIC, HEVC, Takeout, Backblaze, B2, quota, recoverable-trash]
---

# Google Photos 原件清理與備份收尾（2026-10-15）

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

## 完整新備份已完成（2026-10-15 收尾）

- 本節取代 2026-10-13「匯出中、Photos 待完成、舊 B2 保留」的狀態快照；之前的原件清理與容量估算仍是其標示日期的歷史紀錄。全部已授權備份與清理已完成，不因舊 PID、舊 progress 或分項 frozen manifest 的 pending 欄位重跑已完成工作；依使用者要求，監控排程維持 PAUSED。
- 普通影片沿用最高 1080p、不放大、硬體 HEVC Q60；CVSD 若候選更大則依已授權決定保留較小原片，不擅降品質。
- Drive 是 2026-10-12 快照，排除根目錄 Share、垃圾桶與已刪除祖先，保留 Git 內部結構。主封存 45,388,302,310 bytes、42,522 檔／6,625 目錄，完整 B2 SHA256 回讀與逐檔實際還原 SHA256 已通過。主封存 SHA256 為 `8e29b230cba2c607f88502483c9c194f08dba7bcbd29820a1b09f56fcdec8e50`；另有 1,062 份 owned extras、108 份共用檔案補充、三份完整 My Maps KMZ 與三份表單 ZIP 的驗證證據。這不是 2026-10-15 重新抓取的 Drive 即時快照。
- 唯一相簿來源為 2026-10-13 11:15 Asia/Taipei 最新排除 Trash 的匯出，不使用 09:33／09:37 或早先含垃圾桶的作業。三個資料 ZIP 加一個報告 ZIP，共 140,791,207,483 bytes；全部完成完整 B2 SHA256 回讀、ZIP CRC 與逐檔實際還原 CRC／SHA256。第三卷的 20,892 個成員於 2026-10-15 03:23 Asia/Taipei 全數驗證完成，不能再稱只有回讀完成。
- 最終總稽核確認 245 個還原必要物件，原有 249 個資料與分項證據的 B2 物件版本保持不變。根目錄總清單、物件清單、SHA256SUMS、還原說明與清理回條均已上傳並讀回驗證。根目錄 RESTORE-MAPS.json 修正早期 restore_path 的重複 .kmz 副檔名；KMZ 本體未改，早期 frozen 分項清單不重寫。
- 使用者明確授權後，已按精確 fileId／fileName 永久移除 65 個三月舊 B2 版本，共 535,175,554,940 bytes；另取消三組同來源中斷 multipart 上傳，46,841,987,072 bytes，合計 582,017,542,012 bytes（582.02 GB）。版本清單已確認舊 ID 全部不存在，新版資料與無關 GitHub 備份未改；原始 Google Drive／Photos 資料保留。這次授權不延伸為日後刪除其他備份。

## Oracle 暫存與工作樹清理

- 本聊天口述的 Synology 實際指 Oracle/swever，沒有另做 Synology NAS 清理。Oracle 來源 ZIP、partial、B2 回讀與還原暫存均已不存在；002／003 的 state／成功 log 支持 worker 已自動清除副本，其餘兩份僅確認不存在，不推定刪除經過。
- 最終人工檢查額外刪除 0 檔、0 bytes，不能把先前自動清除量重算為本次成果。主 worker 已退出、三個任務鎖已釋放，43 份驗證紀錄／程式／完整 log 的 SHA256 保持不變。2026-10-15 14:43 Asia/Taipei 的可用容量快照約 126.22 GB，不是日後容量保證。
- 舊 HAPI 清理已移除 69 整副本；八個副本只清理 45 個可重建 node_modules 路徑。另 64 組 unpacked 工具、共 192 檔，全部與 tracked 封存參考 SHA256 相同後移除。22 個近期／使用中／claim／dirty／遠端保存證據不足副本保留，未知附帶資料剩約 275 MB。此次範圍之外的工作樹、Minecraft、研究資料與憑證不重清。
- 判斷舊工作樹須查真正 reflog／來源活動；批次改寫 Git 記錄檔 mtime 不等於新工作。整副本需 Git 乾淨、無程序／claim 使用且遠端保存有證據，使用普通 Git-aware 移除，不以 force 繞過唯一修改或 ownership。

## 續傳與驗證的可重用做法

- 原生 B2 multipart 可沿用舊分段，但先核對來源 MD5、完整 SHA1 與每個既有分段 SHA1，只補缺段。multipart complete 只證明上傳完成；後續仍需整卷 B2 SHA256 回讀與全部成員實際還原驗證。
- 全域 singleton 維持非阻塞，單卷採 blocking LOCK_EX；不同卷可上傳／回讀並行，預留最大回讀卷、最大解壓成員與 10 GiB 安全餘量。同卷不得有多個 writer；速度慢但持續前進不重啟。
- 私有操作 Python 用 -I，避免本機 inspect.py 遮蔽標準函式庫而執行非預期副作用。空間釋放用實際 filesystem／df 核對，不能把跨目錄 hardlink 的 du 加總當成實際釋放量。

## 可續接證據位置

本輪工作區為 `<workspace-root>/work/backblaze-refresh-20261012/`，以下相對於該目錄；原始遷移專案為 `<home>/Documents/Codex/2026-09-06/google-drive-api-thread-01a0729b-38d4`。只保存相對路徑與統計；不把登入憑證、媒體身分或私人 URL 複製到共享記憶。

- `photos-coexisting-trash-20261013/`：`plan.json`、`result.json`、`completion-check.json`、`trash-readback.json`、`replacement-survival.json`、`album-readback.json`、`retained-media-size-estimate.json`、`quota-after-cleanup.json`。409 個來源的獨立逐件 journal 已標為 trashed_verified；本次暫存登入檔已移除。
- `takeout-inspection/full-library-audit-20261013/`：`audit-result.json`、`completion-check.json`、`global-original-content-audit.json`、完整 member hashes、B2 保留回讀。此 read-only audit 完成時 Photos 備份仍待處理；不能只用該舊 audit 推論整體完成，整體狀態以 2026-10-15 根目錄總回條為準。
- `full-backup-closeout-receipt-20261015.json`、根目錄 `goal-checkpoint.json` 與 `final-backup/` 的 manifest、objects、RESTORE.txt、RESTORE-MAPS.json、SHA256SUMS：完整備份與清理已完成；新根目錄總回條優先於早期分項 pending 快照。
- `photos-no-trash-backup/b2-manifest-verified-20261015.json` 與各 verified.json：四卷完整回讀／實際還原完成證據；不要重新啟動舊 worker。
- `b2-cleanup-preflight-20261015.json`、`b2-old-cleanup-result-20261015.json`：65 個精確舊版本、三組中斷分段與保留物件的前後身分核對；原始 fileId 保留在私有回條，不公開到共享記憶。
- `oracle-temp-cleanup-20261015/result.json`、`final-backup/oracle-temp-cleanup-20261015.json`、`oracle-hapi-cleanup-result-20261015.json`：Oracle 暫存零殘留、43 份證據保留及舊 HAPI 工作樹清理結果。
