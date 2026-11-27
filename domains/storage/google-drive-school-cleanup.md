---
title: Google Drive 學校講義與學習紀錄整理完成紀錄
scope: domain
status: verified-with-limits
updated: 2026-10-12
tags: [google-drive, school, deduplication, 7z, powerpoint, git, videotoolbox]
---

# 學校講義與學習紀錄整理

- 範圍為 `document/學校講義`、`document/學習紀錄`。共通的 Drive 原位、保留展開 Git／專案、可恢復垃圾桶和 quota 規則見 [十月整理](google-drive-cleanup-202610.md)。使用者已結束本輪，勿重開已完成的壓縮／刪除佇列。
- 去重查大小、完整 hash、文件／程式引用與資料集分區。ZIP 和解壓版要有完整路徑／目錄／CRC／逐檔內容一致證據；資料集原目錄只在授權替代封存完整雲端讀回與還原後移入垃圾桶。Text 封存保留編碼、相對路徑與引用。PDB 刪除即失去該 binary 的舊符號除錯能力，保留 C source 不證明重編已驗。
- OOXML 重新封裝要保存 XML default namespace，ElementTree 改寫 `ns0:Types`／`ns0:Relationships` 曾令 LibreOffice 載入失敗。保持非圖片 bytes，中文字體以該程序的 fontconfig 驗，再做全頁版面 render／回下載。原生 PowerPoint 動畫、PDF 內嵌媒體播放需各自測試。一般影片 1080p HEVC 標準不直接套到文件內嵌播放器。
- JPEG Motion Photo 要判讀實體 MP4、HDR gainmap 與 MPF，不可依標記推定有影片。保留尾部元件時重算 MPF offset，以 ExifTool 與 bitwise／decode 各驗。國防科技 93 張的 Motion Photo 移除是個別授權，最終依原件重做 HEIC Q70，保留檔名主幹、Drive ID、父目錄。HDR gainmap 資訊驗過但非位元組無損，手機原生 HDR 未測。完整品質與文件編碼限制見 [照片壓縮標準](../media/photo-compression-standard.md)。

## 2026-10-12 資料集封存與最終收手

- 電腦視覺 `hw2/hw2_data.7z`、全民國防 `data.7z`、自動控制 `autodrone-2023/datasets.7z` 與 `track_model/datasets.7z` 保存在原父目錄，解壓及逐檔 hash 驗證後才移除原資料夾。早期「hw2 保留解壓目錄」已退役；原始 ZIP 在垃圾桶，非垃圾桶的是新 7z。標註與資料數值未有損轉換，課程程式未重跑。
- `Music-to-Lyrics/Sentence_and_Word_Parsing.7z` 檔案已完整讀回，使用前需解壓；Git 已原樣恢復，非獨立 archive。Custom Vision 僅移除授權的資料與素材，`老師predict程式.py` 保留。逐項位置、數量和收據在固定原版。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/domains/storage/google-drive-school-cleanup.md)
