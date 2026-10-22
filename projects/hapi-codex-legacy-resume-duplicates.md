---
title: HAPI Codex 舊 session resume 重複歷史（#2086）— PR #2099 以 turn-ID baseline 修
scope: projects/hapi-codex-legacy-resume-duplicates
project: hapi
status: active
confidence: high
evidence: 2026-10-22 trace upstream/main 5d8deb201；PR tiann/hapi#2099 head 49448ad，CI 全綠、HAPI Bot 最終 pass（經 4 輪 review）。
created: 2026-10-22
updated: 2026-10-22
tags:
  - hapi
  - codex
  - replay
  - metadata
---

- `runCodex` 已全面改走 shared projection（`cli/src/codex/shared/`）；bind 時 replay 整個 native thread，靠 `codex:<thread>:...` stable localId 讓 hub 去重。舊 launcher 寫的 agent row `local_id` 是 NULL → 全部重插、時間變 resume 當下。
- 修法（PR #2099）：`metadata.codexHistoryBaseline = { turns, before, pending, capture }`，baseline turn 不再 replay（仍 `committed()` user input、仍做 title repair）。
- 判斷 legacy 的坑：bootstrap 會先把 `capabilities.concurrentClients` 改成 true，所以必須在 `bootstrapExistingSession` 的同一次 metadata 寫入裡（`deriveMetadata` hook）從原始 row 判斷並寫 `capture: true` marker，capture 成功才清掉。`concurrentClients` 是 shared launcher 的 provenance 標記（舊 launcher 從沒設過）。
- 新 binding 要把空 baseline 跟 `codexSessionId` 同一次寫，否則中斷的首次 import 會被誤判成 legacy。
- 子 thread 讀失敗 → 放 `pending`，之後用持久化的 `before`（turn.startedAt 秒，legacy rollout 為 null 視為之前）再 capture。
- HAPI Bot 對 metadata 遷移的 crash/restart 窗口抓得很細；類似遷移一開始就要設計「持久化 marker + 可恢復」。
