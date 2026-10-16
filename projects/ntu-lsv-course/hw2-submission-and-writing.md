---
title: LSV HW2 已繳交、檔案驗證與精簡作業慣例
scope: projects/ntu-lsv-course
status: verified-submitted-not-graded
updated: 2026-10-16
---

# HW2 closeout

- 2026-10-16 22:20（Asia/Taipei），COOL 的 EEE5028 Homework 2 顯示 `Submitted!`。只繳交完整七題答案 PDF，檔名 `LSV_HW2_<studentID>.pdf`；程式碼未附交。不要重複提交，除非使用者要求修正或重交。
- 已下載 COOL 提交檔，與本機最終 PDF 逐位元相同：156,195 bytes；SHA-256 `e0ccd5c25e57679131cf6f19a3a52eb4163e2e34490d2253cb5245729c5a915a`。這證明提交內容一致，不代表評分或教師接受結果。
- 正式檔案在 `<course-root>/邏輯合成驗證/`；提交證據為 `LSV_HW2_<studentID>_receipt.json`。現有來源檔仍沿用舊 Q7 命名，工作區連結指向課程資料夾；後續先解析連結、讀目前 .tex，不拿早期 Q7-only PDF 當完整作業。
- Q7 Rust generator 已改為這份作業專用，六組實驗重新執行；不要再描述為「原程式未修改直接沿用」。本次教師允許替代 solver 的授權由使用者提供，不推論成其他作業的通則。

# 編輯慣例

- Keep it simple：保留題目所需推導、圖與結果；CLI 指令、生成器檔名及例行驗證細節留在程式資料夾，不塞進答案正文。
- 集合大小統一寫 `\lvert S\rvert`，例如 `\lvert\mathrm{onset}(f)\rvert`。
- 可並排時文字／表格在左、圖在右，頂端對齊；不強制每題換頁。minipage 前先結束文字段落，避免表格接在句尾、另一欄被擠到下一列。
- 圖附簡短圖說及真正的圖形圖例。短箭頭的虛線可能被箭頭頭部遮住，可縮小箭頭並用顏色輔助區分；不只靠文字寫 dashed/solid。

# COOL 操作

優先讀 `tools/computer-use/ntu-cool-enter-filechooser.md`。本次 click 不開選檔器；建立 `waitForEvent('filechooser')` 後，對最新 AX 的 file input index 執行 `tab.ax.pressKey(index, 'Return')`，再 `chooser.setFiles([absolutePdfPath])`，立即成功。送出用目前 Submit Assignment 的 `locator.press('Enter')`；回執上的下載連結用 `downloadMedia()`，最後 byte comparison。不要重做多輪 click／切換私人瀏覽器才查既有記憶。
