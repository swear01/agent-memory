---
title: "TSRI 製程名單範本（.xls）填寫規範與 Excel BIFF8 自動化"
scope: "domains/eda"
status: active
updated: 2026-10-30
---

# TSRI 製程使用者名單：BIFF8 與年度授權

- 使用該期官方 `auditscoreimport` 工作表，與 EDA 軟體續用表分開。電話須保留 `09xxxxxxxx` 前導 0，電話、ID/UI No.、FAB ID 先設文字格式 `@`。外籍交換生未有 ARC 時，該次可用台灣停留／居留簽證號；新案件查官方當期接受格式。語言和機構地址用一致的課程／單位資料，避免把私人住址任意匯入。
- 保存 BIFF8/OLE `.xls`。用原生 Excel 直接編輯官方範本副本；`xlwt`／`xlutils.copy` 重寫曾剝掉 drawings、樣式和 metadata，不能以 Excel 可開判定 TSRI 匯入可用。Mac Excel 正常重存也可能因 Windows printer PLS record 被替換而減少約 8 KB，從原內容和匯入欄位驗證，勿只比大小。AppleScript 用 `open (POSIX file "<file_path>")` 開實檔，先 `set number format of range … to "@"` 再寫 cell，避免用路徑字串彈出文件選擇阻塞。
- 製程申請代理人可用本人登入進案件協助行政。PDK/CBDK 的 NDA 下載資格仍需列於當年度核准使用者名單，代理權不會代為取得，介面也沒有切換教授身分按鈕。年度法治測驗須本人作答；上一年度通過不沿用，先核對目前製程年度及本人完成狀態。
- 學生看到「查詢製程授權名單」No Permission 不足以判斷個人資格，請看「檢視下線資格」中的「您的製程授權」（英文預設可能收合）。當次 116 年度測驗完成後 U18 已顯示，並非所有案件都保證立即匯入。若被導回 SSO 請走正常登入，憑證異常需先驗官方端點。

[原範本結構、年度測驗及 U18 驗收紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/domains/eda/tsri-process-application.md)
