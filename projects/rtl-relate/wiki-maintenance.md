---
title: AIsimpV GitHub Wiki 的來源與維護
scope: project
project: rtl-relate
status: active
updated: 2026-09-30
---

# AIsimpV Wiki 維護

`swear01/AIsimpV` 的 GitHub Wiki 已啟用，包含 `Home`、`Results-and-Evidence`、`Method-and-Reproduction` 及 `_Sidebar`。Wiki 供人類快速檢查研究狀態；`README.md`、`docs/`、`experiments/` 中的正式報告、task 定義和資料檔才是內容來源。摘要需保留「可驗證性」與「效能」的區別，以及 frontend PASS、property SAFE、certificate ACCEPTED 的證據界線。

後續修改原 repo 文件時，先查看 `AGENTS.md` 指向的 `docs/wiki.md`：若結論、數據解讀、實驗狀態、重跑步驟或導覽變動，同一任務更新對應 Wiki 頁面。Wiki 位於獨立 Git remote `swear01/AIsimpV.wiki.git`；從最新 remote 編輯，推送後檢查公開頁面與連結。僅有文字修正且摘要未受影響時，記錄已檢查、無需更新。

GitHub 空白 Wiki 的第一頁無法由 Git push 或官方 REST/GraphQL API 初始化；啟用 Wiki 後仍須已登入使用者先在網頁儲存 `Home`。此 repo 已完成首次初始化，後續可正常 clone 和 push Wiki remote。這是 2026-09-30 實際遇到並核對 GitHub 文件的限制。
