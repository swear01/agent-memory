---
title: QMD 共享記憶、索引與同步容易踩的坑
scope: global
status: active
created: 2026-08-18
updated: 2026-11-28
tags:
  - shared-memory
  - qmd
  - github
---

## 查詢與索引

- 工具、瀏覽器或 session 第一次非預期失敗時，先查 `qmd search "<服務> <動作> <原始錯誤>" -c memory`，再用 `qmd get` 讀解法和適用版本。缺少觸發條件時，修正擁有流程的 skill 入口。
- 沒有 `-c memory` 的搜尋會混入預設索引中的本機 collections。若同一條規則或結案報告排在共享解法前，先用 `qmd collection list` 核對索引來源。SwairM5 的舊 `codex-memory-notes` 已獨立，僅在需要回查時用 `qmd --index legacy search "<關鍵字>" -c codex-memory-notes`。
- `qmd status` 的 pending 是整個索引的統計。先核對檔案屬於哪個 collection，日常只更新共享向量用 `qmd embed -c memory`。在 QMD 2.8.3，`update` 的 pending 提示可能仍用預設模型計算。以 scoped embed 的 `All content hashes already have embeddings` 或實際向量涵蓋率確認。
- GPU 回報 `Failed to create any embedding context` 時，用 `qmd doctor` 確認裝置與可用 VRAM。GPU 實際失敗後，可用 `QMD_FORCE_CPU=1 QMD_EMBED_PARALLELISM=1 qmd embed -c memory` 單工重試，完成後確認 embedding freshness。有從 Mac 複製同模型向量的無 GPU 機器，依各機流程更新。
- QMD 2.8.3 的 `update` hook 在 collection 根目錄執行。共享庫的 hook 若用 `git pull --ff-only`，遇分歧應先人工釐清，避免索引更新擅自改 Git 歷史。

## 寫入與同步

- 共用內容放進 `swear01/agent-memory` 的 Markdown。保留會改變下一步的症狀、版本、做法與驗證。一次性進度、逐次量測留在原始證據，更新同一個問題的現行筆記。
- Codex 本機 ad-hoc 可含私人資料。只把適合公開的長期經驗整理進共享庫。推送前查遠端可見性與 author、committer，提交使用 GitHub noreply。共用 QMD SQLite、向量和 cache 都可本機重建。
- 使用者已授權每次更新記憶後同步。只提交核對過的檔案，保留其他工作，確認本次提交到達遠端。再跑 `qmd update`、需要時 embed，最後用實際查詢讀回。遠端同步和本機檢索要各自驗證。
- 新檔先 `git add` 再執行 `.githooks/pre-push`，check 基於 `git ls-files`，也檢查 Git 歷史。工作樹和 staged 都跑 `git diff --check`。設定 `core.hooksPath .githooks` 可在 push 時重跑檢查。既有無關失敗須在基準重現；本次新增的問題要先修。
- CI 用 regex 掃私人路徑、信箱和憑證樣式，Markdown 反引號也會命中。範例路徑用 `<remote-home>` 等 placeholder，憑證只寫 `Bearer <api-key>` 等佔位符，個人帳號用角色描述。

## 從歷史 session 提取記憶

- 只在本次授權的範圍讀取已結束或封存的 session，保存分頁、涵蓋範圍與來源雜湊。先移除憑證、PII、private context 和 runtime identifiers，再交付必要的去識別上下文；原始 logs、私密資料和暫存清冊不提交、不索引。找到同一經驗先合併，新的執行結果經獨立查證才寫入。單純沒有找到新教訓，留在批次紀錄即可。
- 以 `active` 記已驗證的長期知識；執行結果待驗的候選應明寫 `needs-review` 以及缺的證據。QMD 快照須來自沒有寫入中的資料庫，並核對嵌入模型相容。原始批次紀錄見 [ingestion ledger](../../archive/global/workflows/memory-ingestion-ledger.txt)。

## 遠端歷史仍含私人資料

覆蓋檔案只能清 working tree。經明確授權做歷史清理時，先備份 refs 和未提交差異到私有且未索引的位置，隔離重寫並逐筆核對受影響的 commit、目標 tree、最新遠端 SHA 與完整 CI。僅更新指定 branch 並以舊 SHA 作 lease。若只改 metadata，先確認工具會否重寫早期簽章；`git filter-branch --env-filter` 曾擴大受影響範圍。

其他 checkout 要在乾淨的遠端 main 上重套本機工作，避免再次推回舊歷史。即使 branch 和 PR refs 都已清乾淨，舊 SHA 仍可能由 GitHub API 或快取讀取。遠端快取清除須另查 GitHub Support 的處理結果。

[完整工作流演進與驗證紀錄](https://github.com/swear01/agent-memory/blob/d1e08b2dad393fd2ecf9dbbee4f69d53bd6d8755/global/workflows/shared-memory.md)
