---
title: Pi Processes 通知切入 tool flow 會讓 Meta 收到重複 output
scope: tool
tool: Pi coding agent
tags: [pi, pi-processes, openai-responses, tool-history, meta]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# Pi Processes 通知切入 tool flow 會讓 Meta 收到重複 output

Pi 0.84.2 的長 session 原先使用 `openai-codex/gpt-5.6-luna`，之後切換成
`meta/muse-spark-1.2-contributor`；Meta Responses API 連續回覆：

```text
Duplicate function_call_output for call_id '...'. Each function_call must have exactly one matching function_call_output
```

## 根因

出錯的歷史順序為：

1. Codex assistant 發出 `bash` tool call。
2. `@aliou/pi-processes` 0.10.9 在真正 tool result 前持久化一筆
   `ad-process:notification`；該筆為 `attention: "ignore"` 的 log-match 通知。
3. Pi core 把 custom message 投影成 `user` context。
4. Pi AI 的 Responses history 正規化器把 user message 視為 tool flow 中斷，先為
   未完成 call 合成 `No result provided` tool result。
5. 真正 tool result 隨後仍被保留，轉換後同一 `call_id` 出現兩筆
   `function_call_output`，Meta 依 Responses 契約回 400。

跨 provider 使舊 Codex tool ID 被正規化成 Meta request 使用的 64 字元 call ID，
但不是重複的直接成因；直接成因是 process notification 切入 call/result 之間，
以及 Pi core 沒有丟棄後到的 unmatched result。

## 已驗證邊界

- 錯誤只出現在該舊 session 的 Meta request；不是 OpenAI Codex OAuth 回覆。
- JSONL 中原始 tool call/result 各只有一筆；第二筆 output 是 request 組裝時合成，
  不是 session 重複落盤。
- 400 中的 call ID 可精確反推到 JSONL 裡的 Codex `call_id|fc_item_id`；Pi 對
  foreign provider ID 做字元替換與 64 字元截斷後，得到 Meta 拒絕的 ID。
- 同機、同 provider、同 model 的全新無狀態呼叫成功：

  ```bash
  pi --provider meta --model muse-spark-1.2-contributor --no-session -p 'Reply with exactly: OK'
  ```

  退出碼為 0 並回覆 `OK`。

因此故障不是帳號、Meta model、`pi-meta-oauth` 或原始 JSONL 重複。Meta endpoint
只是拒絕 Pi 組出的非法 Responses payload。

## 處理方式

已污染的舊 session 改用新 session；不要手改 JSONL 或刪除認證。

預防新事件可更新 `@aliou/pi-processes`：0.12.0 已把 `context`／`ignore` 通知從
`deliverAs: "steer"` 改成 `"nextTurn"`，其原始碼明示這是為了避免通知切開 tool
call 與 result。這項變更會防止相同通知順序再次形成，但不會修復已落盤的舊
session history。

Pi AI 0.84.4 的相關 `transform-messages.js` 與 0.84.2 無差異，因此只升級 Pi
core 不會修掉這條路徑。`pi-codex-conversion` 3.0.23 的相關 message-history 邏輯
也與 3.0.15 無差異；Meta 使用 Pi core 的普通 `openai-responses` provider，並不
走 Codex adapter 的去重路徑。
