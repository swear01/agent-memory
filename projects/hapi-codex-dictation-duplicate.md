---
title: HAPI Codex 語音輸入送出的訊息會在 hub 存成兩筆
scope: projects/hapi-codex-dictation-duplicate
project: hapi
status: active
confidence: high
evidence: 2026-10-18 讀取正式 hub v0.30.7.4 的 session messages、對照原始碼追蹤，並在 PR #1436 修正與測試。
created: 2026-10-18
updated: 2026-10-18
tags:
  - hapi
  - codex
  - dictation
  - localId
---

# 現象

v0.30.7.4 維護版中，Codex session 用 HAPI 語音輸入（dictation 直接送出）的每一則訊息都會在聊天室顯示兩次。
Claude session 不受影響。Codex 只執行一次（每對重複訊息後面只有一個 `turn_complete`），所以是重複「顯示／儲存」，不是重複執行。

hub 內的資料形狀：

- 第一筆：`sentFrom: webapp`、`deliveryMode: queue`、`localId: null`、`invokedAt` 在插入時就被蓋上。
- 第二筆：緊接的下一個 seq，`sentFrom: cli`、`localId` 是新的 UUID、文字完全相同。

# 根因鏈

1. PR #1436（swear01 自己的 open 上游 PR，`feat/web-persist-voice-input-session-switch`，在維護版中以 `PERSONAL_PR_POLICY_EXCEPTION` carry）
   的語音自動送出走 `useDictation.ts` → `api.sendMessage(sid, msg, null, ...)`，**不帶 localId**，也沒有樂觀泡泡。
   一般打字送出走 `useSendMessage`，會帶 `localId`。
2. 所有 Codex session 都是 shared Codex（`cli/src/codex/runCodex.ts` 直接 export `runSharedCodex`）。
   `cli/src/codex/shared/root.ts` 的 `onUserMessage` 用 `localId ?? randomUUID()` 當成 Codex `clientUserMessageId`。
3. Codex 回報 `userMessage` item 時，`cli/src/codex/shared/projection.ts` 用 `item.clientId` 呼叫
   `sendUserMessage(text, undefined, id)`，把使用者訊息回傳給 hub。
4. hub `store/messages.ts` `addMessage` 只在 `(session_id, local_id)` 已存在時去重。
   打字訊息的 clientId 就是原本的 localId → 去重成功；語音訊息原 row 的 `local_id` 是 NULL、回傳帶隨機 UUID → 去重失敗 → 第二筆。

上游 main 沒有 #1436，網頁語音不會觸發；但第 2 步的漏洞仍在，任何不帶 localId 的送訊來源（例如 Telegram 回覆）打到 Codex session 都會一樣重複。

# 修正狀態

- 2026-10-18 已修在 PR #1436 本身（使用者授權直接更新該 PR；上游沒有等價的語音直接送出功能）：
  `web/src/lib/dictationSend.ts` 新增 `sendVoiceMessage`，用 `makeClientSideId('local')` 補 localId；
  `useDictation`、`useRealtimeDictation`、`HappyComposer` 語音送出、`SessionChat` 跨 session 語音工具送出都改走它。
  PR head `45bf446`（含 merge upstream/main），root typecheck 與 Web 全套 3,439 tests 通過。
- 維護版要等下一次 rebuild carry 新的 #1436 head 才會生效；已部署的 v0.30.7.4 仍會重複。
- 未修：CLI `shared/root.ts` 的 `localId ?? randomUUID()` 洞仍在上游，其他不帶 localId 的來源（例如 Telegram 回覆）打到 Codex session 仍會重複；
  根治候選是 hub `messageService.sendMessage` 對缺 localId 的使用者訊息補一個。

# 診斷方法

用 CLI token 換 web JWT 後，以 `GET /api/sessions/<id>/messages` 比對 user 訊息的 `localId`、`meta.sentFrom`、`seq`；
`localId: null` 的 webapp 訊息後面緊跟一筆同文字 `sentFrom: cli` 的訊息，就是這個問題。
