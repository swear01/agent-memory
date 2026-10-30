---
title: LSV HW3 寫完卻漏交、遲交補救與排版慣例
scope: projects/ntu-lsv-course
status: verified-submitted-late-not-graded
created: 2026-10-31
updated: 2026-10-31
tags:
  - ntu-cool
  - late-submission
  - quine-mccluskey
---

# 發生了什麼

- HW3（COOL course 65093、assignment 425307）期限 2026-10-30 23:59（Asia/Taipei）。答案 PDF 早在 2026-10-27 就完成並編譯，但那次 session 結束時沒有繳交步驟，也沒有提醒，結果漏交。
- 2026-10-31 00:35:59 用 Canvas 三步上傳 API 補交，COOL 回 `late: true`、`seconds_late: 2160`（36 分鐘）。下載回來與本機 PDF 逐位元相同（82,896 bytes），回執存成 `LSV_HW3_<studentID>_receipt.json`。
- Syllabus 規定遲交每天扣 20%。助教信箱寫在 syllabus 首頁（`課程教材/原始講義/LSV26 syllabus.pdf`）。用 `himalaya -a ntu`（只能寄）寄出兩封信：先致歉並附 PDF，補交後再請求免扣 20%。是否免扣由助教決定，結果未知。

# 教訓

- **作業寫完不等於交了。** 完成作業 PDF 的 session 要以「已繳交並驗證」或「已設提醒」收尾；只有使用者明確同意才代為繳交，但至少要在收尾時明說「還沒交，期限是 X」。
- 遲交求情信不要寫「期限前就已完成」：如果期限後還改過送出的版本，這句話經不起比對。照實寫延遲分鐘數（以 COOL 的 `seconds_late` 為準，不要用估計值）。

# 答案排版慣例（使用者 review 後的要求）

- Quine-McCluskey 的「pairwise merging」要列出合併過程：先依 1 的個數分組列出帶 output tag 的 minterm，再用三欄表列出每對合併；tag 為 `00` 被拒絕的合併另起一行列出，不放在表格裡（否則欄寬會被撐開）。
- Boolean matrix（column covering）用全格線細框（`|l|*{9}{c|}`、`\arrayrulewidth=0.3pt`），f/g 輸出之間用雙線分隔，這樣 1 才容易對到欄位；不要用 booktabs 只畫橫線。
- Cyclic core 寫成兩張小表：「只有一個 1 的列 → essential prime」、「選中的 prime → 涵蓋的列」，最後一句結論。
- Prove/disprove 題（例如 unate 題）證明保持一個論證、一次結論，不要再用額外段落或講義引述重講一遍。
- AI disclosure：使用者要求寫明所有題目全部使用 AI、模型 Claude Opus 5.5、reasoning effort（以 session metadata 查實際值）；語音輸入的 prompt 改寫成摘要。
