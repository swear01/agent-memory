---
title: HAPI CLI 原本不能改既有 session 的 permission mode；hapi set-mode 在 PR #2095
scope: projects/hapi-cli-set-mode
project: hapi
status: active
confidence: high
evidence: 2026-10-22 查 upstream main 5d8deb201 的 cli/src/commands/registry.ts（沒有 mode 指令）；PR tiann/hapi#2095 CI 全綠、HAPI Bot clean，等維護者 merge（swear01 沒有 push 權限）。
created: 2026-10-22
updated: 2026-10-22
tags:
  - hapi
  - cli
  - permission-mode
---

- upstream main 上，permission mode 只能在啟動時用 `--permission-mode` / `--yolo` 指定；之後要改只能用 web，或直接打 hub 的 `POST /api/sessions/:id/permission-mode`（先 `POST /api/auth` 用 CLI token 換 JWT）。
- Issue tiann/hapi#2094 → PR #2095 新增 `hapi set-mode <id-or-prefix> <mode>`，重用 `cli/src/modules/pingPeer/pingPeer.ts` 的 auth 和 session 解析。
- 陷阱：hub 的 permission-mode route 只依 flavor 驗證。shared（`capabilities.concurrentClients`）Codex session 若被存成 `safe-yolo`，下次 resume 會 fail（`sharedLaunchConfig` 會 throw）。web picker 有過濾，但 route 原本沒擋；#2095 在 hub route 和 CLI 都加了 guard。2026-10-26：分支 `fix/codex-drop-safe-yolo`（262a40eba）把 safe-yolo 從 Codex catalog 整個移除，這些 guard 也一併刪掉；upstream PR 尚未開。
- 有意不做 MCP tool：提升 peer 的 mode 屬於權限提升，留在 agent 的 shell approval 之後（參考 #1402 / #1535）。
