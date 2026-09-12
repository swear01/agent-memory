---
title: 改名／替換任務只改目標項目，清單類設定必須保留其他項目
scope: global
status: active
confidence: high
evidence: 2026-09-11 至 2026-09-12 DeepSeek model id 改名時，Pi 的三層 allowlist 被過度收斂到只剩兩個模型，連 Codex/Meta/Qwen 一起移除；使用者退回並要求還原（PR #46 → 更正 PR #47）。
created: 2026-09-12
updated: 2026-09-12
tags:
  - scope
  - config
  - allowlist
  - migration
  - review
  - renamed
  - fleet
---

# 規則

當任務是「把 X 改名／換成 Y」時：

- 清單類設定（`allowed` / `enabledModels` / `whitelist` / filter `rules` / allow list / route 表 / `models.allow`）
  **只改指向 X 的項目**，其他項目原封不動。
- 不要順手「收斂範圍」、「清掉不再需要的」或「讓選單變乾淨」。使用者沒說的移除一律不做；要移除先問。
- 即使元件設計上就是「只允許白名單」，白名單的**內容**是既有政策，不是這次任務的標的。
- 變更後用「集合差異」自我檢查：diff 應該只有目標項目改變。例如 `enabledModels` 8 項 → 8 項（其中 2 項改名），
  **不是** 8 項 → 2 項。
- Fleet 級操作先在單機驗證「其他項目都還在」，再推全部；覆蓋範圍會放大錯誤。

# 為什麼（實際案例）

2026-09-11 DeepSeek 把官方 model id 改成 `deepseek-flash`。我在把 fleet 的 DeepSeek id 從
`deepseek-v4-flash` / `deepseek-v4-pro` 換成新名的同時，把 Pi 的三層 allowlist（`settings.json` 的
`enabledModels`、`model-filter.json`、`extensions/strict-model-allowlist.ts`）一起收斂成「只剩兩個
DeepSeek id」。結果 `openai-codex/*`（Luna / Sol / Terra / Daybreak / Astra）、
`meta/muse-spark-1.2-contributor`、`valkyrie-ninfer/qwen3.8-27b` 全部從 HAPI 選單消失。使用者指出
「應該要保留原本用的其他模型…只替換 deepseek」，隔天從各機 `backup-*-pre-deepseek-rename/` 備份還原
（詳見 `tools/pi/deepseek-model-id-migration.md`）。

兩個容易誤判的地方：

1. **HAPI 的模型選單來源是 `strict-model-allowlist.ts` 過濾後的 runtime registry**
   （`get_available_models` → `ModelRuntime.getAvailableSnapshot()`），不是 `settings.json` 的
   `enabledModels`（那只有本機 TUI `/model` scope 在用）。「想讓選單只剩 X」的念頭會直接導致動到 allowlist。
2. 「舊 id 之後會被官方移除，所以順便清掉」是**推論**、不是需求；使用者講的只有改名。

# 自我檢查問句

- 這次 diff 有沒有刪掉任何**不是**任務目標的項目？
- 使用者只說了「改名」嗎？那「只留 X」是誰的主張？
- 若是 fleet 級變更，單機驗證「其他項目還在」的證據在哪？
- 命名為「allowlist / 白名單 / filter」的檔案，我是改了它的**內容指向**，還是改了它的**範圍**？
