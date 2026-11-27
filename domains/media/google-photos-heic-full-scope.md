---
title: Google Photos 全量 HEIC 範圍與 API 批次回驗流程
scope: domains/media
status: active
updated: 2026-09-15
tags: [Google Photos, Takeout, HEIC, Drive API, rclone, SHA256, Motion Photo, in-app-browser]
---

# Google Photos 全庫來源與 HEIC 逐批回驗

1. 全庫來源 Takeout 選所有年份、所有相簿，確認控制項已載入並回讀 Google 建立匯出成功。2 GB ZIP 分卷或磁碟限制只改下載／轉檔／回驗批量，不縮使用者指定範圍。使用指定的內建 browser，延用登入及實際提供的重新驗證流程，不占用鼠標、不改安全設定或自動恢復停用排程。
2. 照片走 [HEIC Q70](photo-compression-standard.md)，保留來源、JSON、拍攝時間、位置、ICC 和 HDR。Motion Photo 是 image＋影片／聲音／XMP，Live Photo 是配對檔；普通靜態重編不能證明動態保留，要求保留時需 Photos 回傳播放驗證。不要把大型影片專案的排除照片範圍移植到全庫照片工作。
3. Library API 上傳用原檔名主幹＋`.HEIC`，落盤來源 SHA、上傳 SHA、mediaItem ID 和原相簿映射；回讀未知結果，勿盲重送。API 上傳批量與 Takeout 次數分開。多批上傳及 API 回讀完成後，精確選那些驗證相簿做一次 Photos Takeout 到 Drive，同帳號以授權 Drive API/rclone 下載。匯出可能有快照延遲，按相簿和 SHA 驗聯集，逐卷查 Drive 大小/MD5、ZIP CRC、HEIC SHA。直接寫 Drive 的原檔不算 Photos 回驗。
4. Library/Picker `baseUrl=d` 不保證原容器，也可能缺位置 EXIF。一張實測 API 回 PNG、Takeout 回到原 HEIC SHA，不能外推其餘照片。遇內建 browser `Invalid InterceptionId`，查既有 Takeout→Drive API 路徑，不能承諾升級或反覆重試有效。Takeout 建立本身仍可能要求近期驗證；不用 Web 分卷下載可減少點擊，不取消安全挑戰。Data Portability 過往未能代替 Photos Takeout，需另核產品及帳號資格。
5. 清理前對原件和替代品的精確雲端身分、相簿／最愛／封存、日期與 live 存在性。Takeout URL、mediaItem ID、mediaKey、dedupKey 不能互代。檔名／hash 相同仍可能有多個雲端項目。ICC 原始 bytes 或缺省狀態嚴格保留；輕微實測日期差僅依已授權容差接受，8 小時差不能按牆鐘相同視為同一 instant。
6. Library API 無原件二進位覆寫或照片 delete。已核准的網頁批次 MoveToTrash 使用實際查回的 dedupKey，未公開接口批量先重驗；parser 展開同一行所有 frames，以 ID 集合核對，避免把 250 responses 錯看成 225。每批先寫意圖和 receipts，未知結果先回讀。最終核對精確垃圾桶差集，確認替代品仍在主圖庫，不清空垃圾桶。暫存登入資訊不留在 Git 或記憶，失效從授權入口續接。
7. 清 Drive 暫存同時查 Takeout 和 staging。FileProvider 顯示來源不等於本機獨立副本；驗完整本機 ZIP/MD5，保留清單再清精確遠端項目。`rclone lsjson --stat` 的虛擬根可能無 ID，改用父層列表核對。全量影像詳細來源與 2026-09-15 留存例外見固定版本；另見 [2026-10-15 收尾](google-photos-b2-cleanup-20261013.md)。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/domains/media/google-photos-heic-full-scope.md)
