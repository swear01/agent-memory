---
title: HAPI wrapper parity 快照要用 CI 釘住的 Bun 版本重錄
scope: tools/hapi/wrapper-parity-goldens
status: active
confidence: high
updated: 2026-11-27
---

HAPI 的 `cli/src/parity/corpus/claude/*/golden.ts.json` 記錄 CLI 註冊的 RPC 與送到 Hub 的 agent state。新增 RPC 或 state 欄位會改到這些快照，即使 `bun run gen:fixtures` 顯示 `shared/fixtures` 沒有變動。

從 repo 根目錄用 `package.json` 中的 Bun 版本跑 `bun run parity:record`，檢查快照差異，再跑 `bun run test:parity`。要沿用對照組中的標準化 UUID。新增 UUID 欄位可能讓後續的 `<uuid:n>` 重新編號。

[tiann/hapi#2623](https://github.com/tiann/hapi/pull/2623) 實證：Bun 1.3.14 重錄時出現無關的子程序環境欄位差異。改用 CI 的 Bun 1.4.4 後，18 份 Claude 快照只有新 RPC、回合欄位和 UUID 編號差異。完整 parity 49 項通過。
