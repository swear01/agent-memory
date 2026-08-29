---
title: Pi 的 OpenAI Codex OAuth 登入與驗證
scope: tool
tool: Pi coding agent
tags: [pi, openai-codex, oauth, login]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# Pi 的 OpenAI Codex OAuth 登入與驗證

Pi 的 ChatGPT／Codex 訂閱登入 provider 是 `openai-codex`，不是 `openai`、
`google` 或 `meta`。Pi 0.84.2 沒有不帶 provider 的通用 auth status；以下命令
會直接失敗：

```bash
pi auth status
pi auth list
```

## 重新登入

啟動 Pi，在互動介面執行：

```text
/login openai-codex
```

無圖形瀏覽器的主機選 `Device code login (headless)`，由使用者在另一個瀏覽器
完成 OpenAI 裝置授權。不要保存或轉錄 OAuth token、裝置驗證碼或帳號資訊。

## 驗證

先檢查 provider readiness：

```bash
pi auth check --provider openai-codex --json
```

成功結果應包含 `"status":"ready"` 與 `"authType":"oauth"`。登入完成訊息本身
不足以證明模型可用；再執行一次最小端到端呼叫：

```bash
pi --provider openai-codex --model gpt-5.6-luna --no-session -p 'Reply with exactly: OK'
```

2026-08-30 實測修復前 `pi auth check` 回報 `invalid_state`；重新執行上述登入後，
readiness 變成 `ready`，模型呼叫退出碼為 0 並回覆 `OK`。
