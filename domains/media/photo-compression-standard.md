---
title: 照片與 PDF／PPTX 內嵌圖片的統一壓縮標準
scope: domain
status: user-confirmed
updated: 2026-10-11
tags: [photo, compression, HEIC, Q70, PDF, PPTX, quality]
---

## 使用者確認的預設

2026-10-11 使用者明確要求沿用先前照片標準，並套用到 PDF／PPTX 內嵌圖片的壓縮方針。獨立照片預設採既有 **HEIC Q70** 流程，保留原解析度；不要自行改成 JPEG Q90／Q92，或把舊專案 HEIF Q15、Q50、Q80 的個別試驗當成目前通用標準。執行前先核對原流程的實際編碼器與參數，不能只看副檔名或品質數字。

保留日期、EXIF／ICC、HDR、透明度等適用資訊；不要為節省容量默默移除功能或縮圖。動態部分是否保留依具體授權：本次國防科技 93 張照片已獲授權移除 Motion Photo 影片，這不是所有照片可一律移除動態或 HDR 的授權。

## PDF／PPTX 內嵌圖片

同樣先以既有照片標準作為品質與壓縮方針，不因圖片在文件裡就自行選較高的 JPEG 品質。HEIC Q70 與 JPEG Q70／Q90 不是可直接換算的品質；尚未實測的替代參數不能標為等效或符合標準。

容器與閱讀器限制要先查證並說明，再採最小可行方案：PDF 1.7 的標準影像 filters 不含 HEIC，不能把影像串流換成 HEIC 卻沿用 `/DCTDecode`。PowerPoint 能匯入 HEIC 也不代表直接改 PPTX 封裝後能在所有閱讀器顯示，須查實際儲存格式並渲染／開啟驗證。無法直接沿用時，明列原因、可用編碼、實測品質與容量，不能默默改成另一套標準。文字、向量、線稿、透明圖片與可編輯結構仍須保留。

盡量從保留的原始來源重做，避免對已經有損壓縮的版本反覆轉碼。保留簡潔檔名主幹與原資料夾位置；改格式時使用真實副檔名及 MIME，檢查程式／文件引用。上傳後完整讀回比對，驗證成功後才依已授權範圍清來源。

## 本次偏差與更正狀態

先前國防照片實際使用 JPEG Q90；PPTX 流程使用 JPEG Q92、4:4:4、長邊最多 2560；PDF 流程從 JPEG Q88／Q92／Q95 選擇，依內容降到 200／300 dpi。這些是歷史實作，不是使用者確認的新標準，也未驗證與 HEIC Q70 等效。

原因是 agent 為保留原解析度、HDR／動態容器以及文件可讀性，自行採取保守 JPEG 方案，卻未先核對既有標準或說明差異。這是流程疏漏；不能歸因於使用者要求改標準，亦未證明 HEIC 做不到。

93 張照片的動態影片後來已移除：621,707,529 → 363,959,922 bytes，靜態像素與 HDR gainmap 不變，全部雲端讀回通過。這仍是 JPEG Q90。本次更新記憶並核對舊參數，**未完成 HEIC Q70 重轉，也未重新替換 PDF／PPTX**；後續不能把記憶更新當成媒體已重做。回條見 `<workspace-root>/outputs/國防照片移除動態影片_20261011.json`，工作區為 `<user-documents>/Codex/2026-10-08/new-chat`。

## 格式依據

- [Adobe PDF 32000-1:2008](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf)：7.4 的標準 filters 與影像編碼。
- [Microsoft HEIF extension codec](https://learn.microsoft.com/en-us/windows/win32/wic/heif-codec)：HEIF／HEIC 與系統 codec；不能由系統能解碼推論所有 PPTX 閱讀器支援。
