---
title: Pi 跨 provider 延續 session 可能產生重複 tool output
scope: tool
tool: Pi coding agent
tags: [pi, openai-responses, tool-history, provider-switch]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# Pi 跨 provider 延續 session 可能產生重複 tool output

Pi 0.84.2 的長 session 原先使用 `openai-codex/gpt-5.6-luna`，包含大量平行
tool calls；Codex OAuth 失效後，在同一條 session history 切換成
`meta/muse-spark-1.2-contributor`，Meta Responses API 連續回覆：

```text
Duplicate function_call_output for call_id '...'. Each function_call must have exactly one matching function_call_output
```

## 已驗證邊界

- 錯誤只出現在該舊 session 的 Meta request；不是 OpenAI Codex OAuth 回覆。
- JSONL 中沒有重複的完整 `toolCallId`，以 `|` 前的 Responses `call_id` 正規化
  後也沒有重複；錯誤中的 call ID 只出現在伺服器 400，沒有落盤的原始
  tool call/result。
- 同機、同 provider、同 model 的全新無狀態呼叫成功：

  ```bash
  pi --provider meta --model muse-spark-1.2-contributor --no-session -p 'Reply with exactly: OK'
  ```

  退出碼為 0 並回覆 `OK`。

因此故障位於舊 session 跨 provider 重播時的 Responses payload/history 組裝，
不是帳號、Meta model 或原始 JSONL 重複。官方 Responses function-calling 契約要求
每個 `function_call` 只追加一個同 `call_id` 的 `function_call_output`。

## 處理方式

不要在帶有既有 tool history 的 Codex session 直接切換到 Meta；為新 provider
建立新 session。保留舊 session 作唯讀診斷，不要手改 JSONL 或刪除認證。

當時安裝版本為 Pi 0.84.2、`@howaboua/pi-codex-conversion` 3.0.15、
`pi-meta-oauth` 0.4.4；查詢時前兩者已有 0.84.4 與 3.0.23，是否已修正此特定
跨 provider history 問題尚未驗證，不應把升級當成已確認修復。
