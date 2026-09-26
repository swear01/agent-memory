---
title: "TSRI 製程名單範本（.xls）填寫規範與 Excel BIFF8 自動化"
scope: "domains/eda"
status: active
updated: 2026-09-26
---

# TSRI 製程名單範本（.xls）填寫規範與 Excel BIFF8 自動化

## 背景與情境

申請 TSRI（國家實驗研究院台灣半導體研究中心）EDA 軟體與製程（如 U18、C18、T18 等）權限時，需填寫 TSRI 官方提供的 process namelist 匯入名單（工作表名稱：`auditscoreimport`），完成後寄交審核以開通學生帳號權限。

## 核心規則與填寫規範

1. **外籍交換生身分證號（ID/UI No.）**：
   - 交換生在台期間若尚未取得居留證（ARC / UI No.），可使用中華民國停留/居留簽證號碼（Visa Number，格式如 `115KULxxxxxx`、`115SINxxxxxx`）填寫。

2. **電話格式（Mobile）**：
   - 所有在台聯絡電話統一為 `09xxxxxxxx` 十碼格式，去掉國際碼（如 `+886`），並保留前導 `0`。

3. **儲存格型別要求（文字格式 `@`）**：
   - 電話、身分證號、FAB ID 欄位必須在 Excel 中指定為文字格式（`@`），避免 Excel 將開頭為 `0` 的電話自動轉為數值導致 `0` 遺失，或將英數字號誤判。

4. **學校（School）與地址（Address）語言與慣例**：
   - 整張工作表語系須保持一致（建議統一為繁體中文）。
   - **地址統一慣例**：TSRI 官方範本中的所有範例列皆統一填寫同一個單位地址（如竹科園區地址）。在學期課程名單中，實務上全班統一填寫開課教授之實驗室/研究室地址（例如學校系館/教學館），既符合 TSRI 以學校機構為存檔單位的審核慣例，亦避免學生個別填寫戶籍、租屋處所造成的格式混亂或缺漏。

## Excel BIFF8 檔案處理與自動化陷阱

1. **避免 Python `xlwt`/`xlutils` 重新寫入**：
   - TSRI 範本為舊版 OLE 複合二進位檔案（BIFF8 `.xls`）。
   - 使用 Python `xlutils.copy` 或 `xlwt` 重新寫入時，只會產生最陽春的 BIFF 結構，會自動剝離原檔中的 Drawing 圖層、樣式表、列印配置與 metadata，導致 39 KB 的範本縮減至 ~10 KB，易觸發 TSRI 自動化匯入系統報錯。
   - **正確解法**：在 macOS 上應透過 AppleScript 自動化調用原生 Microsoft Excel 直接以原範本進行原地編輯並儲存。

2. **檔案大小從 39 KB 變為 32.8 KB 的成因**：
   - 原版 TSRI 範本內含一筆 8,224 bytes 的 `0x004D`（`PLS`, Print Environment）二進位記錄，係當初製作範本的 Windows 電腦殘留的 HP 網路印表機驅動暫存。
   - 在 Mac Microsoft Excel 開啟並存檔時，Excel 自動清除無效的 Windows 印表機驅動暫存，並替換為 30-byte 標準通用列印設定（減少約 8.2 KB）；扣除後加上新增學生的列資料、SST（Shared String Table）與儲存格參照，在 OLE 512-byte Sector 對齊下淨減少 12 個 Sector（6,144 bytes），產出標準大小為 32,768 bytes（約 32.8 KB），為正常二進位最佳化而非資料缺失。

3. **AppleScript 操作 Excel 指令要點**：
   - 開啟檔案路徑必須使用 `POSIX file` 轉換：`open (POSIX file "<posix_path>")`；若誤用字串直接傳入 `workbook file name` 會彈出 Finder 檔案對話框導致腳本阻塞。
   - 範圍格式設定與儲存：
     ```applescript
     tell application "Microsoft Excel"
         open (POSIX file "<file_path>")
         tell active sheet of active workbook
             set number format of range "A21:G33" to "@"
             set value of range "..." to "..."
         end tell
         save active workbook
         close active workbook
     end tell
     ```

## 代理人機制與製程技術文件下載權限

1. **代理人身分不等於製程下載權限**：
   - TSRI 系統的「製程申請代理人」（每位教授上限 2 名）僅具備行政協助功能（代填申請、代為聯繫、管理 EDA Cloud 帳號等）。
   - 製程技術文件（PDK / CBDK）受晶圓廠嚴格 NDA 具名授權約束，不適用代理人權限自動繼承。
   - 即使具備代理人身分，其帳號、身分證號與 Email 仍必須獨立出現在當年度核准的「製程使用者名單 Excel」（`auditscoreimport`）並由 TSRI 人工審核匯入，技術資料下載專區（`ncms.tsri.niar.org.tw`）才會解鎖。

2. **法治素養宣導測驗前置要求**：
   - 自 113 年度起，申請人及被授權使用者皆須每年通過線上「法治素養宣導測驗」（直達選單：`menu=16`）。未通過測驗者，系統即使收到名單也會鎖定下載權限。

3. **無獨立切換身分按鈕**：
   - TSRI 系統無「切換為教授身分」按鈕。代理權限係於代理人登入個人帳號後，進入「晶片製作 → 製程/矽智財申請」時由後台權限自動帶入教授名下案件。
