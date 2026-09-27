---
title: Codex spawn-peer 將預留的新 session 誤當 cold resume
scope: tools/hapi
status: verified
updated: 2026-09-27
---

HAPI `v0.30.7.2` 的 `spawn-peer --agent codex` 可在 runner 在線時立即失敗，回報 `Existing HAPI session has no Codex thread binding`。swairM5 的失敗呼叫回傳 HTTP 502，mazu runner stderr 亦有相同錯誤；這與後續機器離線或 Google 授權失效是不同問題。

上游範圍已核實：`tiann/hapi` 的 `main` `86c88df9` 尚無 `spawnSessionWithRemitOnce` 與 `spawnRemitOperation`；這兩者來自尚未合併的 PR #1771（head `95558c3a`），由維護版的 `497604e7` carry。這是該功能與 shared Codex runtime 的整合缺口，不能宣稱純上游 main 已提供並重現相同 atomic spawn 路徑。追蹤 issue 為 `tiann/hapi#1928`。#1922 的另一種 spawn-peer 實作採 spawn→ping→verify，並非這次出錯的原子操作。

貢獻流程：使用者要求先在 `tiann/hapi` 查重並開 issue，再提交上游修正 PR；個人 fork 的 `swear01/hapi#27` 不能代替上游交付。準備 PR 前必須核對真正的 upstream 和未合併功能相依，不能只因現有 clone 沒有 upstream remote 就把 fork/main 當成上游基準。

根因是 `hub/src/sync/syncEngine.ts` 的 `spawnSessionWithRemitOnce` 先建立尚無 `codexSessionId` 的 `spawn-with-remit` 紀錄，再經 `spawnSession` 與 runner 傳成 `--existing-session-id`。`cli/src/codex/shared/frontend.ts` 的 `runSharedCodex` 將它當成 cold resume，要求已存在的 native thread binding，因此在啟動 shared runtime 前拋錯。

不能只刪除 frontend guard：該版 `shared/runtime.ts` 的新 thread 路徑另建 HAPI row，無法保留 remit 預留的 row identity。修正須完整區分「接管預留的新 row」與「恢復有 native thread 的舊 row」，並沿 Hub、runner、CLI/bootstrap 維持同一 session ID。

`v0.30.7.3` 雖新增 `reservedSessionId`，一般 machine spawn 會走新 adopt-stub 流程，但 `spawnSessionWithRemitOnce` 仍把預留 row 傳入 `existingSessionId`；`spawnSession` 只在沒有既有 ID 且提供 namespace 時設為 preallocated。不能據此宣稱升級已修好 `spawn-peer`。

驗證：以 GitHub release tag 的原始函式做隔離執行，兩版 frontend 都拒絕無 native binding 的 remit row；新版 Hub `spawnSession` 仍將該 ID 放入 existingSessionId，而 reservedSessionId 為 undefined。這是來源函式的隔離重現，沒有聲稱新版已在真實 Mac 上重現，也未部署修復。Pi 的成功不代表 Codex 啟動流程已恢復。

修正已在 `swear01/hapi#27` 實作並驗證：frontend 僅允許 runner 啟動、pending remit、沒有 hostPid、未 archived 的新 row 在沒有 native binding 時繼續；runtime 初始 `create` 將 `initialOptions.existingSessionId` 傳給既有 bootstrap。後續 `/new` 與 fork 不帶初始選項，因此另建 row，不會重用原 remit identity。Hub 的 `preserveHubOwnedMetadata` 保留 spawnRemitOperation，毋須放寬 machine-spawn stub 的 adopt 規則。

驗證包含新增入口案例先紅後綠、CLI typecheck、18 個入口／真實 Codex runtime 檢查，以及完全隔離的真實 Hub→Runner→Codex `spawn-peer`、`wait-peer`、同 remit retry、`archive-peer`。模型端使用本機 mock Responses，沒有付費推理。這些證據不等於正式 runner 已部署更新。

測試邊界：opt-in `runtime.integration.test.ts` 的 MockSession 必須實作 EventEmitter，否則會在 `registerControls` 因缺少 `on` 而失敗。較大的 `frontend.integration.test.ts` 在修正前後皆於 `secondary attach to terminal-owned engine` 逾時；未修改基準 `e80ab76f` 已重現，不能把它當成 remit 啟動失敗或本次已修好的功能。
