---
title: LSV HW3 寫完卻漏交、遲交補救與排版慣例
scope: projects/ntu-lsv-course
status: verified-submitted-late-not-graded
created: 2026-10-31
updated: 2026-11-03
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

# 檔案放哪裡（2026-10-31 整理後）

- 課程雲端資料夾 `邏輯合成驗證/` 頂層放最終交付物：`LSV_HW3_<studentID>.pdf/.tex`（2026-11-03 起是交件後修正的 v4，未重傳 COOL；實際交出的 v3 只在 Google Drive 版本紀錄裡，receipt 的 sha256 已對不上本機 PDF）、`hw3.pdf`（題目）、`..._receipt.json`、`..._receipt_0035.json`。
- 工作資料放 `lsv/<hw|pa>/`；HW3 的 `lsv/hw3/` 只放被取代的 00:35 首次繳交版（`..._0035.pdf/.tex`）與 `ai_disclosure_log.md`。
- Review 過程的中間檔（Codex/Gemini 輸出、v2/v3 草稿、submit prompt）不要存；不要在 Documents 另開 `LSV_HW3_review_*` 資料夾，見 [[keep-home-directory-clean]]。

# 交件後修正：Q4 對角線慣例（2026-11-03）

- 交出的 v3 把老師的講法寫成「把 j=i 代進公式」，結論變成「只有 S1 一定是等價關係」。這是 v2 那次 AI 改寫時對老師意思的誤讀，當時沒有回頭對逐字稿。
- 老師在 10/27 HW3 檢討（`課程教材/影片解析/第08週_10-27/2_.../transcript.en.txt`，約 05:30、09:00、17:00、18:20）的說法是：i=j 就是把變數換成自己、不做任何取反，所以 (x_i,x_i) 一律在每個 S_k 裡；公式裡的取反只對 i≠j 有定義。
- 依此慣例的正解：S1、S3–S8 是等價關係；S2 自反、對稱但不遞移（反例 f = x̄1·x2·x̄3，兩次 S2 互換組合起來負負得正，只得到 S1）。v4 已改成單一結論，AI disclosure 加 [G] 記錄這次修改。
- 教訓：任何「老師說／instructor's reading」的寫法，都要回到 `影片解析/<週>/transcript.en.txt` 找原句再寫；作業檢討課的逐字稿比講義更能決定題意慣例。
- 編譯：在 scratchpad 複製 tex 後 `xelatex` 兩次（Fandol 字型不存在時 tex 會自動退回 Noto Sans CJK TC），只把 PDF 複製回 Drive，避免 aux/log 留在雲端資料夾。
