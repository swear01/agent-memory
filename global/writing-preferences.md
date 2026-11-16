---
title: "文章與研究報告的寫作偏好：清楚命名、程式碼比較與自然正文"
scope: global
status: active
updated: 2026-11-17
tags:
  - article-writing
  - research-report
  - writing-preferences
  - gemini
---

# 文章與研究報告的寫作偏好

2026-09-23，使用者在整理 RTL 研究報告時指出文字太有 AI 感，要求使用讀者看得懂的詞，並明確要求原始與簡化設計的比較使用 code block。之後授權完整重寫 `article-writing`，讓這些改善適用後續文章與報告。

- 用具體名稱說明對象與測量內容，不直接拿內部代號當表格標題。例如測量確實對應時，將「直接 C」與「A property」改為「原始設計驗證時間」與「簡化後設計驗證時間」。必要術語仍保留並說明，程式識別字不能為了好讀而任意改名。
- 程式差異用可選取的 code block 或 diff，清楚標示改寫前後；圖片適合另外呈現架構或數據，不用來代替要求的程式碼比較。
- 報告直接呈現問題、工作與證據。依題目安排段落，避免固定模板、重複摘要與空泛標題；未被要求時，不混入報告時間分配、口頭講法、預設問答或寫作教學。
- 降低 AI 感要處理模糊命名、結構與重複，不只替換形容詞。保留數據的單位與計算範圍，區分完成結果、解釋、限制及下一步，不能因潤飾而誇大成果。

完整規則以 `<skillshare-source>/article-writing/SKILL.md` 為準，重寫已由 `shared-skills` PR #35 合併。這份記憶保存偏好與理由，不複製整份 skill；來源追蹤的處理見 [自有 skill 的來源切換](../tools/skillshare/locally-maintained-skill.md)。

## 給人讀的文字最後交給 Gemini 修訂

2026-11-17，使用者明確要求記住：投影片、報告、文章等給人閱讀的文字，交付前要交給 Gemini 做最後修訂。使用者認為 Gemini 最擅長寫散文，是工作流程中重要的一環。

- 交給 Gemini 前先鎖定引文、事實與數字；修訂後 Claude 要對照原稿 diff，確認這些內容沒有被改壞，再交付。
- `GEMINI_API_KEY` 用 `vault` 取得（例如 `vault run GEMINI_API_KEY=<item> -- cmd`），不要印出或寫進檔案。
