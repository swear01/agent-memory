---
title: "HAPI archived session 先 Reopen 再送 peer 訊息"
scope: tools/hapi
status: active
updated: 2026-09-29
---

# Archived session 的 peer 訊息順序

對已封存的 coding session 直接執行 `hapi ping-peer`，可能得到 `resume_failed: Session webhook timeout`；這不能證明對話內容遺失或 session 無法接手。先用 HAPI 的 **Reopen** 功能恢復該 session，確認 `active: true`，再送 peer 訊息。2026-09-29 的一次交接在 Reopen 後成功取得原 agent 的完整摘要。

當時 CLI 的 `hapi resume` 明確拒絕 Hub archived session，並要求先做 Hub Reopen。診斷時應先區分 Hub 封存狀態與底層 Codex 對話檔封存；後者另見 `codex-archived-reopen-path.md`。不要把首次 webhook timeout 當成最終阻塞，也不要盲目重試 peer 訊息。
