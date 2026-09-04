---
title: Word DOCX 行距裁切與 DriveFS 同步驗證
scope: tools
status: active
confidence: high
created: 2026-09-04
updated: 2026-09-04
tags:
  - docx
  - word
  - python-docx
  - libreoffice
  - drivefs
sources:
  - verified template-based DOCX assembly and PDF export
---

# Word DOCX 行距裁切與 DriveFS 同步驗證

## 大字沿用固定行高會被 Word 裁切

- 若範本的 `Normal` style 為固定行高，例如 11 pt 內文搭配 `13.5 pt EXACTLY`，封面或表格中的 15.5–26 pt 文字若仍使用 `Normal`，就會繼承不足的固定行高。Microsoft Word 可能裁掉字形上下緣，即使其他 renderer 看起來勉強可讀。
- 不要為了修封面而全域改掉 `Normal`。只在封面標題及資料表儲存格段落明確設定 `line_spacing = 1.2`，並確認 OOXML 是 `w:lineRule="auto"`，讓行高隨字級擴張。
- 表格列不要設定固定高度；保留自動展開，並提供足夠的 cell margin。可用 `python-docx` 驗證 `paragraph_format.line_spacing_rule == MULTIPLE` 且 `row.height is None`。

## 全頁圖片也要隔離段落行距

- Inline image 所在段落若繼承內文的固定行高，LibreOffice 或 Word 可能把全頁圖片移出可視範圍或造成裁切。
- 圖片段落應獨立設定：取消 snap-to-grid、段前段後為 0、使用單行或倍數行距，再插入圖片。不要依賴 `Normal` 的固定行高。

## 版面修正的回歸證據

- DOCX XML 可讀不代表版面正確。每次版面修改後都要重新輸出 PDF／PNG，逐頁檢查裁切、重疊與分頁。
- 若只修封面，可比較修正前後第 2 頁到最後一頁的 rendered page hashes；全部相同可證明修改沒有影響主文與附件。
- 從最終 DOCX 匯出 PDF 後，以 `pdfinfo` 確認頁數與紙張尺寸，以 `pdffonts` 確認字型已嵌入，並仍要逐頁視覺檢查。

## Google DriveFS 覆蓋後的同步邊界

- 將檔案建立或覆蓋到本機 Google Drive 同步資料夾後，DriveFS 的 item-id extended attribute 可能短暫消失再出現。
- 先比對來源與目的檔 SHA-256、確認檔案能重新開啟，再輪詢 `com.google.drivefs.item-id#S` 是否恢復；不要在 attribute 尚未出現時宣稱已同步，也不要輸出實際 item ID。
