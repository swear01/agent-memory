---
title: Google Photos Q70 原件清理、完整 B2 備份與 Oracle 暫存收尾
scope: domains/media
status: verified-closeout
updated: 2026-10-15
tags: [Google Photos, Q70, HEIC, HEVC, Takeout, Backblaze, B2, quota, recoverable-trash]
---

# Photos 清理與 B2 的可恢復性

- Google Photos 並存原件依「目前 live 正確身分＋已驗替代品」核對。上傳回條只證明上傳，還要回驗、核對相簿、確定原件在雲端而非只清本機。Takeout 相簿重複匯出、supplemental JSON、同檔名異內容、同內容異雲端身分都不能直接當 duplicate。書寫 intent，未知結果先回讀，最後驗精確 trash 差集和替代品留在主庫。以原批次處理小日期、相簿、尺寸差異的授權不外推到其他素材。
- 檔案總大小、非垃圾桶內容估計和實際帳戶配額分開。免計配額與上傳政策有關，不能由拍攝年份推定。批次移入垃圾桶的 bytes 不等同即時釋放；未公開 storage tuple 只能當分析線索，需重讀帳戶容量。
- B2 native multipart 可沿用分段，先驗來源 MD5、完整 SHA1、每段 SHA1，只補缺段。全域 nonblocking singleton 及 per-volume blocking lock，隔離不同卷的 writer，預留最大回讀卷、最大解壓件及 10 GiB。慢但有進度不要重啟。Python 啟動用 `-I`，避免同目錄 `inspect.py` 遮蔽 stdlib。實際釋放量用 filesystem/df，別加總跨目錄 hardlink 的 du。
- Multipart complete 之後再做各物件全量回讀 SHA、整卷 archive test 和實際還原逐檔 CRC／SHA。取代舊 B2 僅按已授權精確 fileId/fileName，分開取消未完成上傳。事前事後比對物件清單，留新版本和無關備份。傳輸驗證、完全恢復與刪除權限分開。

## 2026-10-15 結案與恢復入口

- 2026-10-12 Drive snapshot 已驗實際還原，排除根 Share、垃圾桶和已刪祖先，保留 Git；owned extras、共用檔、My Maps 和 Forms 另有分項清單。Photos 只使用 2026-10-13 11:15 台北時間排除 Trash 的匯出，四卷 B2 完整讀回、ZIP CRC 與逐檔還原均通過。不是任意時點的即時快照或全庫 HEIC 完成證據。
- 根目錄 `full-backup-closeout-receipt-20261015.json`、`final-backup/` 的 objects／SHA256SUMS／RESTORE.txt／RESTORE-MAPS.json 為恢復入口，優先於舊 frozen manifests 的 pending 欄位。還原 My Maps 時看根修正的 `restore_path`，早期清單有重複 `.kmz` 副檔名。
- 對已授權 409 項驗精確 Trash／替代品，沒有清空 Google 垃圾桶。舊 B2 版本與 multipart 舊工作已收尾、監控維持 PAUSED。當時「Synology」口述是 Oracle，沒有對 NAS 清理。Oracle 已用逐項 Git／reflog／遠端保存／程序引用判斷舊 HAPI 工作樹，保留 dirty、claim、使用中和無獨立保存證據者，不因 mtime 或 node_modules 佔用就刪來源。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/domains/media/google-photos-b2-cleanup-20261013.md)
