---
title: 替換只改指定目標，停用與封存必須核對範圍
scope: global
status: active
confidence: high
evidence: 2026-09-11 至 2026-09-12 DeepSeek model id 改名時，Pi 的三層 allowlist 被過度收斂到只剩兩個模型，連 Codex/Meta/Qwen 一起移除；使用者退回並要求還原（PR #46 → 更正 PR #47）。
created: 2026-09-12
updated: 2026-11-04
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

> 關鍵字（方便搜尋）：改名 rename、替換 replace、只換 X only rename one item、保留其他模型 keep other models、
> 不要順手移除 don't remove unrelated entries、清單類設定 list-type config、allowlist 白名單、enabledModels、
> whitelist、filter rules、models.allow、range 收斂 scope creep / over-narrowing、fleet 級變更 blast radius。

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


# 停用／封存前核對範圍

2026-11-04，使用者要用現成 CLIProxyAPI 替換自製 Gateway，Swear Review 應繼續使用新 Gateway。Agent 卻把相近名稱與含糊措辭解讀成連 Review 都退役，封存 repo、停止服務／tunnel、改掉 review 規則與文件。使用者糾正後已復原。這是範圍誤解，不是 Swear Review 故障，也不是使用者要求停用。

下次執行方式：

1. 先以實際 repo／服務名稱核對「替換對象、保留的呼叫端、明確要停用的對象」，每個停用動作都要能對應到使用者的明確要求。替換 Gateway、資料庫或其他基礎元件，不能推導成停用依賴它的應用程式。
2. 發現名稱近似、語音轉錄含糊，或「繼續用」與「archive」的對象不清楚時，先完成不會中斷服務的設定／查證，再只問一個具體的範圍問題；未釐清前保留相關服務與資料。範圍清楚且已獲授權的例行工作直接繼續。
3. 先讓保留的呼叫端實際經過新元件完成工作，再停用明確被替換的舊元件。驗證要同時證明「目標已換新」與「依賴服務仍正常」，不能只看新 Gateway healthz 或模型清單。

本案目前意圖與驗證紀錄見 [Gateway 與 Swear Review 記憶](../../projects/deepseek-latch-gateway/cliproxyapi-retirement.md)。錯誤結論要從 active 記憶／文件中改正；歷史紀錄應標明是 agent 誤解，不能改寫成 owner 授權退役。QMD 是索引：修改 canonical Markdown 後才 update/embed，並 search/get 回讀新版內容。
