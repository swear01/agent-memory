---
title: Happy、HAPI 與原生 CLI 的執行中訊息交付
scope: tools/agent-collaboration
status: active
confidence: high
updated: 2026-11-27
evidence: Claude Code 和 Codex 官方文件、兩個上游的固定版本原始碼及 GitHub issue 與 PR。
tags:
  - agent-protocol
  - happy
  - hapi
  - claude-code
  - codex
redaction: passed
---

# 執行中訊息交付

訊息可以先由介面或 Hub 收下。Agent 是否在本輪讀到，要看目前的交付意圖、後端支援和可用的工具邊界。寫入 stdin、RPC 回覆、Agent 讀入是不同時點。送出結果不明時，以原本的訊息 ID 對帳，避免重複執行。

## 原生 CLI

- Claude Code 的一般 Enter 先收下訊息。若工具仍在跑，普通文字可在工具完成後進入同一輪，剩餘訊息則由後續輪次處理。命令和 shell 命令有各自的排程規則。[Claude Code 互動模式](https://code.claude.com/docs/en/interactive-mode#when-claude-code-sends-what-you-queued)
- Codex CLI 忙碌時，預設 Enter 可 Steer，Tab 排到下一輪。Codex Remote 介面提供預設交付方式的選項。[Codex CLI 維護者說明](https://github.com/openai/codex/issues/11530) [Codex Remote 指南](https://developers.openai.com/blog/mastering-codex-remote-for-engineering)

## Slopus Happy

- Happy CLI 的 Claude remote 在 `result` 後才拉下一筆，Codex wrapper 也未接 `turn/steer`。手機忙碌時送出 follow-up 預設排到下一輪。[Claude remote，4a21b91e](https://github.com/slopus/happy/blob/4a21b91e451044970e4a731220af2beb69bbd024/packages/happy-cli/src/claude/claudeRemote.ts#L337-L374) [Codex app-server client，4a21b91e](https://github.com/slopus/happy/blob/4a21b91e451044970e4a731220af2beb69bbd024/packages/happy-cli/src/codex/codexAppServerClient.ts#L1126-L1192)
- 原生 iOS 的送出會先進裝置 outbox，Hub 收件後則儲存、發出更新並回覆 HTTP。具備 receipt capability 的 session，iOS encoder 也會送 `expectsAcceptance`、`queuedWhileBusy`。[iOS outbox](https://github.com/slopus/happy/blob/4a21b91e451044970e4a731220af2beb69bbd024/packages/happy-ios/Sources/HappyCore/Chat/Sending/OutboxSender.swift#L104-L116) [iOS metadata](https://github.com/slopus/happy/blob/4a21b91e451044970e4a731220af2beb69bbd024/packages/happy-ios/Sources/HappyCore/Chat/Sending/UserMessageEncoder.swift#L37-L67) [Hub v3 收件](https://github.com/slopus/happy/blob/4a21b91e451044970e4a731220af2beb69bbd024/packages/happy-server/sources/app/api/routes/v3SessionRoutes.ts#L181-L238)
- Happy Agent daemon 的遠端 follow-up 走 Steer。若將兩者放在同個選單，應依 session 類型、capability 和缺省值送出。#1868 是 legacy CLI 的提案，#1910 與 happy-agent#57 處理新版 Agent。這些 PR 在 2026-11-27 均未合併，舊 React Native app 路徑也已從 main 移除。後續行動端工作以原生 iOS、Android 為準。[上游討論 #1867](https://github.com/slopus/happy/issues/1867) [#1868](https://github.com/slopus/happy/pull/1868) [#1910](https://github.com/slopus/happy/pull/1910) [happy-agent#57](https://github.com/slopus/happy-agent/pull/57) [舊 app 移除](https://github.com/slopus/happy/commit/e46a907d269ac87e3c1836e9ff1f9f48e87cbc01)
- #1388 是本機 Claude 終端接管卡住的個案。#2088 則回報 iOS 介面無法分辨已收件、排隊和開始執行，使用者因而可能重送。[#1388](https://github.com/slopus/happy/issues/1388) [#2088](https://github.com/slopus/happy/issues/2088)

## tiann/HAPI

- Web 一般 Send 固定走 Queue，Claude、Codex 都已有逐則 Steer。#1466 與 #1480 說明 Pi 也曾刻意改為 Queue 優先。#1979（@rapind）是 Codex session 的 Queue／Steer 偏好提案，2026-11-27 仍開啟且有衝突。[Web 路徑，6f7420aa](https://github.com/tiann/hapi/blob/6f7420aa6ec87878391fa0b4535711ba04e0b0ba/web/src/lib/messageDelivery.ts#L43-L63) [#1466](https://github.com/tiann/hapi/issues/1466) [#1979](https://github.com/tiann/hapi/pull/1979)
- Hub 一般訊息的 `deliveryMode: 'steer'` 目前只保留給 Pi。Codex 和 Claude 的 Web 偏好可以沿用先建立同一筆 queue row、再用原 `localId` 呼叫逐則 Steer 的做法。上游 `ping_peer` 的 `auto` 已用這條路徑服務具備可插話 turn 的 peer，#2298 已合併。[Hub 正規化，6f7420aa](https://github.com/tiann/hapi/blob/6f7420aa6ec87878391fa0b4535711ba04e0b0ba/hub/src/sync/messageService.ts#L173-L183) [Peer 交付，6f7420aa](https://github.com/tiann/hapi/blob/6f7420aa6ec87878391fa0b4535711ba04e0b0ba/cli/src/modules/pingPeer/pingPeer.ts#L800-L816) [#2298](https://github.com/tiann/hapi/pull/2298)
- 自動插話需在送出時綁定目標 turn，並在 dispatch 時驗證。現有 Codex 手動 Steer 是收到請求才抓取當時的 turn。兩次 HTTP 呼叫間若換輪，應把原訊息留在佇列，避免改到另一輪。[Codex RPC handler，6f7420aa](https://github.com/tiann/hapi/blob/6f7420aa6ec87878391fa0b4535711ba04e0b0ba/cli/src/codex/shared/root.ts#L702-L707)

### 對齊 Claude Code 工具邊界

- 使用者指定 `tiann/hapi` 為此需求的上游。普通文字應在安全的工具批次邊界讓 agent 於同一輪讀到，錯過該輪則沿用原佇列。待審批或提問時先等決定，普通 Send 不觸發手動 Steer 的拒絕待審批功能。[Claude Code 工具邊界](https://code.claude.com/docs/en/interactive-mode#when-claude-code-sends-what-you-queued) [HAPI 手動 Steer #2432](https://github.com/tiann/hapi/pull/2432)
- 上游 [#2623](https://github.com/tiann/hapi/pull/2623) 的 `7278fe12` 已讓 Claude remote 和 Codex 共用 Hub 偏好、原 `localId` 與 runner 端的 turn 和待決策檢查。一般 Send 省略客戶端觀察到的 turn。定向 Hub 測試確認，缺 token 時會嘗試插入 Hub 當下的輪次，附過期 token 則保留原訊息排隊。Codex 的 `turn/steer` RPC 確認與原生 user message 開始事件也是不同時點。[Hub 判斷](https://github.com/tiann/hapi/blob/7278fe12/hub/src/sync/syncEngine.ts#L2060-L2089) [Codex RPC 確認](https://github.com/tiann/hapi/blob/7278fe12/cli/src/codex/shared/queue.ts#L710-L717)
- Pi 0.58.4 起等當前工具批次完成，但 HAPI 的 Pi RPC 成功早於原生讀入。OpenCode v2 的 inbox 分開回報 `enqueued`、`delivered`，現有 native POST 只帶訊息 ID，仍需驗證送出跨輪。Cursor ACP 和新版本 Antigravity ACP 有手動 Steer，但一般回覆完成才可確認的路徑無法證明讀入邊界。[Pi 0.58.4](https://github.com/earendil-works/pi/releases/tag/v0.58.4) [OpenCode v2 回執](https://github.com/tiann/hapi/blob/6f7420aa6/cli/src/opencode/opencodeRemoteLauncher.ts#L2072-L2118) [Cursor #2229](https://github.com/tiann/hapi/pull/2229) [Antigravity 回執](https://github.com/tiann/hapi/blob/6f7420aa6/cli/src/agy/agyAcpRemoteLauncher.ts#L1554-L1604)
