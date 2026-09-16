---
title: "預設測試的成本不應暗中取決於本機資料是否 hydrate"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: e2188205341ca6e614348be5644fe20e9ee96c97575909b600f9936913556e98
---

# 預設測試的成本不應暗中取決於本機資料是否 hydrate

歷史量測回報 registry checker 在已有 DVC evidence 的工作區會逐檔 hash，單項占完整 suite 約八成；乾淨 CI 未拉資料，通常不走相同重型分支。

把快速 schema／metadata 檢查與完整 evidence audit 的責任明列，開發測試可分層，但正式 promotion 仍須完整 hash、tamper、symlink 與路徑界線驗證。若同次 audit 共用 hash，須保持檔案身份與修改邊界。來源沒有實作拆分，單次時間不作持續效能保證；無關的 cost-only 訊息另留原紀錄。
