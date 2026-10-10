---
title: Google Drive 學校講義與學習紀錄整理完成紀錄
scope: domain
status: verified-with-limits
updated: 2026-10-10
tags: [google-drive, school, deduplication, 7z, powerpoint, git, videotoolbox]
---

## 範圍與使用者偏好

整理範圍為 Drive `document/學校講義`、`document/學習紀錄`。保留簡潔的原始名稱與資料夾結構；同內容選一份保留，不加長括號名稱。壓縮後直接更新原檔，保留 Drive ID、名稱及父目錄，不在旁邊留另一份新版。移除用可恢復垃圾桶；本次未永久刪除，也未清空垃圾桶。

以下是 2026-10-09 至 10-10 的已驗證操作紀錄，不能當成往後的即時容量或繼續刪除授權。證據工作區 `<workspace-root>` 為 Mac 的 `<home>/Documents/Codex/2026-10-08/new-chat`。

## 2026-10-10 去重及文字封存

- 394 個內容重複檔案移入 Drive 垃圾桶，其中 375 張照片／圖片；重複檔案合計 544,956,235 bytes，照片占 229,624,976 bytes。既有檔案改名數為 0。
- 以新讀回的大小及 MD5 確認內容重複，並檢查受影響文件與程式引用；保留訓練／測試分區需要的副本及不同用途的輸出素材，不能只憑全域雜湊相同就刪除。
- `學習紀錄/碩一/國防科技/期中報告` 的報告引用 20 張 `media/` 圖片，因此保留 `media/`，只移除相同的 `raw/` 副本。
- 保留完整的 `學校講義/大一/工程數學-微分方程(一)`；另一個 `微分方程` 目錄的 12 份重複 PDF 及清空後的資料夾移入垃圾桶。
- `學習紀錄/高中/高三資專/文字資料.7z`：168 份 CSV／JSON 等資料，**2,501,940,467 → 392,888,034 bytes**。封存內保留原相對路徑，使用前解壓至高三資專根目錄；封存後未重跑依賴這些資料的程式或訓練流程。
- 參數：`7zz a -t7z -mx=9 -m0=LZMA2:d=128m:fb=273 -ms=on -mmt=4 文字資料.7z .`。已完整解壓並逐份比對 168 個 SHA256；上傳後完整下載雲端壓縮包，MD5、SHA256 與 `7zz t` 皆通過，才將原始散檔移入垃圾桶。
- 壓縮包 SHA256：`254233a87210d797ed9232a7b7685fe74ccc2f14b8f0b5ddc37106dae318ef22`。
- 本批共 **562 個檔案**進 Drive 垃圾桶（394 重複 + 168 封存來源），另有一個空資料夾；非垃圾桶檔案減少 **2,654,008,668 bytes（約 2.65 GB）**。垃圾桶仍占配額，不能宣稱帳戶總容量已釋放。
- 本機下載來源、解壓驗證資料及雲端讀回壓縮包移入 macOS 垃圾桶；保留 `<workspace-root>/work/school-next-cleanup-20261010/文字資料.7z`、清單及回條。

## Git：只稽核，全部保留

12 個 Git repository 的 `.git` 合計 535,153,443 bytes。8 個已確認分支／標籤尖端被遠端涵蓋；另 2 個未設 remote（全民國防、正規方法 Final 簡報），高三資專有本機分支未能證明遠端涵蓋，舊 DSA 課程遠端 SSH 連線逾時。

所有 `.git` 均保留，沒有替這些課程 repository 推送。遠端分支涵蓋不等於 reflog、未引用物件、暫存區等本機歷史都已備份；後續處理前仍需獨立備份與驗證。電腦視覺的次分支後來已透過遠端 ancestry 比對確認，屬於上述 8 個，不再列作未涵蓋。

## 2026-10-10 簡報：原位取代完成

4 份 PPTX 以原 Drive ID 更新，原名稱、父目錄及連結保留，未建立旁邊的新版；合計 **459,588,766 → 335,488,891 bytes**，減少 124,099,875 bytes。

| 原檔 | 原始 bytes | 更新 bytes | 投影片數 |
| --- | ---: | ---: | ---: |
| 資訊地科場簡報合併版0517.pptx | 179,697,073 | 161,412,857 | 305 |
| CustomVision-Workshop_zh-Hant.pptx | 104,864,061 | 35,792,165 | 46 |
| 楊老師實驗室參訪簡報.pptx | 115,427,051 | 105,313,201 | 16 |
| 生物第五組.pptx | 59,600,581 | 32,970,668 | 86 |

JPEG Q92、4:4:4、最長邊 2560；透明／平面 PNG 保留，GIF、影片、音訊及投影片 XML 等不變。453 頁全部渲染比較，封裝與雲端完整讀回通過；未以原生 PowerPoint 驗證動畫或影片播放。

OOXML 處理教訓：ElementTree 把根節點寫成 `ns0:Types`／`ns0:Relationships` 曾使 LibreOffice 無法載入，序列化需保留正確預設 namespace。中文字型渲染用該程序的 `FONTCONFIG_FILE=/opt/homebrew/etc/fonts/fonts.conf`，未改全域設定。

電腦視覺 `hw2_data.zip` 的 7z 測試僅在本機進行，**沒有上傳或取代雲端 ZIP**；不要把測試壓縮包當成已完成的雲端整理。

## 2026-10-09 環境與影片

- 6 個 Python 虛擬環境 1,960,498,828 bytes，改以 2,293,538 bytes 的重建資訊 ZIP 保留；502 個封存項目、7 種同 Python minor／較新 patch 的重建及 dependency checks 通過。53,788 個原環境項目已逐項核對後進垃圾桶；未重跑課程作業，不能保證每個原始 micro 版本都已實測。
- 14 支影片候選中 8 支縮小後同 ID 原位更新，6 支未縮小而保留原件；減少 1,230,334,250 bytes。沿用 `hevc_videotoolbox` Q60、B frames 2、最大 1080p、不放大、停用軟體 fallback；完整解碼及 frame timing 驗證通過。

## 證據與續作限制

主要回條均在 `<workspace-root>/outputs/`：`school-cleanup-result-20261010.json`、`school-text-archive-result-20261010.json`、`school-git-audit-20261010.json`、`school-presentation-replacement-20261010.json`、`cv-environment-cleanup-result-20261009.json`、`school-video-compression-result-20261009.json`。

最新本機清單是當天盤點基線加上本次已驗證操作，沒有重新列舉無關雲端變更；若要現在的兩資料夾容量，重新盤點。原檔進垃圾桶與簡報同 ID 更新也不代表舊版本或帳戶配額已清除。不要重跑已完成佇列，下一輪刪除需依新的明確範圍執行。
