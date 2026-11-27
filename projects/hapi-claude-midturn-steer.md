---
title: HAPI 的 Claude 手動 Steer 與一般送出的排隊行為
scope: projects/hapi-claude-midturn-steer
project: hapi
status: active
confidence: high
evidence: HAPI v0.36.7 和 upstream 6f7420aa6 原始碼、定向測試、Claude Code 2.1.302 stream-json 實測；2026-11-28 GitHub 讀回 #2259、#1981、#2229 皆已合併。
created: 2026-10-21
updated: 2026-11-28
tags:
  - hapi
  - claude
  - steer
  - stream-json
---

- HAPI v0.36.7 的一般 Send 走 queue，忙碌等當輪結束。Claude/Codex 的逐則 Steer 走既有 RPC，直接將 Send 的 `deliveryMode` 改為 `steer` 會被 Hub 正規化成 queue。[上游 Claude 手動 Steer #2259](https://github.com/tiann/hapi/pull/2259) 已合併。[v0.36.7 路徑](https://github.com/tiann/hapi/blob/v0.36.7/web/src/lib/messageDelivery.ts#L43-L63)
- Claude CLI 一般 Enter 先收訊息，遇下個 tool boundary 才可能同輪讀到。Claude Code 2.1.302 的 stream-json 若在最後回覆才送，可能在第一筆 `result` 後另開 follow-up turn，wrapper 不宜只用第一筆 `result` 宣告 ready。Codex CLI Enter 是 Steer，Tab 排下一輪。[Claude CLI](https://code.claude.com/docs/en/interactive-mode#when-claude-code-sends-what-you-queued) [Codex CLI](https://learn.chatgpt.com/docs/developer-commands?surface=cli#interactive-shortcuts)
- 區分進 Hub、RPC 接受與 agent 讀入。Claude remote 等 prompt UUID 的讀入回音才報 `Steered`，該版 Codex 在 RPC 接受時即回報，早於原生 user message。[Codex 回執](https://github.com/tiann/hapi/blob/6f7420aa6/cli/src/codex/shared/queue.ts#L710-L725)
- Steer 正待確認時暫停重按、編輯與取消，保留跨 reload pending。ACP abort 的 `stopReason: cancelled` 以 indeterminate 處理，勿自動重播未確認訊息。[上游 #1981](https://github.com/tiann/hapi/pull/1981) [上游 #2229](https://github.com/tiann/hapi/pull/2229)
- 版本結論以 PR／source 為界，排查實際站點要核對部署的 CLI、Hub 與 Web。
