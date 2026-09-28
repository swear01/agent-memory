---
title: Word DOCX 行距裁切與 DriveFS 同步驗證
scope: tools
status: active
confidence: high
created: 2026-09-04
updated: 2026-09-29
tags:
  - docx
  - word
  - python-docx
  - libreoffice
  - drivefs
  - google-drive
  - resource-deadlock
  - on-demand-files
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

## DriveFS on-demand 檔案讀取死鎖（`Resource deadlock avoided`）

- 本機 Google Drive 同步（Drive for desktop）的 **on-demand / 未水合**檔案，用 `open().read()`、`cat`、`cp`、`ditto`、`dd` 讀取時可能全部 `OSError: [Errno 11] Resource deadlock avoided`，即使 `stat` 顯示完整大小、其他小檔案讀得到。
- 這是 macOS FileProvider 對**大檔案 pread** 的已知問題（本輪 76MB PDF 卡住，300KB 的檔案正常），跟網路無關。
- 修法：
  1. **用 Preview 開啟該檔案**（`open -a Preview <檔案>`）強制觸發完整水合，約 20–30 秒後就能正常讀。
  2. 或用能正常讀的 Python（不同 interpreter 時而定）讀；同一台機上 `cp`/`cat` 卡住但某個 python `read()` 成功、或反過來，都見過。
  3. 重啟 Google Drive app（`pkill -9 -f "Google Drive"` 後 `open -a`）不一定立刻好，file provider 要重新註冊，且重啟後大檔案可能更卡——優先用 Preview 水合。
  4. 水合後**先 `cp` 到本地（如 `/tmp`）再處理**，後續操作都對本地副本做，別再碰 DriveFS 路徑。
- 判定：`file <檔案>` 或 `dd` 直接回 `Resource deadlock avoided` 就是 DriveFS 沒水合，不是檔案真的壞掉。

## 直接以 XML 編輯既有 DOCX 範本的防損毀原則

- 不要使用 Python 標準庫 `xml.etree.ElementTree` 解析並寫回 `word/document.xml`。ElementTree 預設會重寫 root `<w:document>` 的命名空間宣告（例如丟失 `mc:Ignorable`、將預設命名空間冠上 `ns0:` 前綴），且無法保障 OOXML 規範中 `<w:pPr>`、`<w:rPr>` 等子元素的嚴格先後順序，這會立即觸發 Microsoft Word 開啟時的「Word 找到無法讀取的內容」損毀阻擋提示。
- 正確做法：使用基於 `lxml` 的 `python-docx` 進行就地修改。針對範本中已有之佔位符或下劃線（如 `_____:_____` 或空格 run），直接修改 `run.text = '...'`，完整保留既有 run 的所有字型、字級、樣式屬性與 schema 順序。
- 若儲存格段落原本無 run 需新增文字，使用 `p.add_run(...)`，不可使用 `p.text = ''` 清除段落（python-docx 的 `p.text = ''` 會留下無 text 子節點的空 run `<w:r/>`，在部分嚴格版 Word 中被判定為格式異常）。
- 驗證方法：解開 docx 比對修改前後的 `word/document.xml`，確認 diff 僅為 `<w:t>` 內的純文字替換，其餘標籤與屬性完全 100% 一致。
