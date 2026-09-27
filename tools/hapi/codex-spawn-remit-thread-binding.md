---
title: Codex spawn-peer 將預留的新 session 誤當 cold resume
scope: tools/hapi
status: verified
updated: 2026-09-27
---

HAPI `v0.30.7.2` 的 `spawn-peer --agent codex` 可在 runner 在線時立即失敗，回報 `Existing HAPI session has no Codex thread binding`。swairM5 的失敗呼叫回傳 HTTP 502，mazu runner stderr 亦有相同錯誤；這與後續機器離線或 Google 授權失效是不同問題。

根因是 `hub/src/sync/syncEngine.ts` 的 `spawnSessionWithRemitOnce` 先建立尚無 `codexSessionId` 的 `spawn-with-remit` 紀錄，再經 `spawnSession` 與 runner 傳成 `--existing-session-id`。`cli/src/codex/shared/frontend.ts` 的 `runSharedCodex` 將它當成 cold resume，要求已存在的 native thread binding，因此在啟動 shared runtime 前拋錯。

不能只刪除 frontend guard：該版 `shared/runtime.ts` 的新 thread 路徑另建 HAPI row，無法保留 remit 預留的 row identity。修正須完整區分「接管預留的新 row」與「恢復有 native thread 的舊 row」，並沿 Hub、runner、CLI/bootstrap 維持同一 session ID。

`v0.30.7.3` 雖新增 `reservedSessionId`，一般 machine spawn 會走新 adopt-stub 流程，但 `spawnSessionWithRemitOnce` 仍把預留 row 傳入 `existingSessionId`；`spawnSession` 只在沒有既有 ID 且提供 namespace 時設為 preallocated。不能據此宣稱升級已修好 `spawn-peer`。

驗證：以 GitHub release tag 的原始函式做隔離執行，兩版 frontend 都拒絕無 native binding 的 remit row；新版 Hub `spawnSession` 仍將該 ID 放入 existingSessionId，而 reservedSessionId 為 undefined。這是來源函式的隔離重現，沒有聲稱新版已在真實 Mac 上重現，也未部署修復。Pi 的成功不代表 Codex 啟動流程已恢復。
