---
title: Antigravity IDE agentapi 喚醒既有對話
scope: tools/antigravity
tool: Antigravity IDE language_server agentapi
status: active
confidence: high
evidence: >-
  2026-10-31 在本機 Antigravity IDE 實測：language_server 同時聽兩個
  127.0.0.1 port，其中一個連線會被對方直接關掉，另一個回
  missing CSRF token。對後者設定 ANTIGRAVITY_LS_ADDRESS 與
  ANTIGRAVITY_CSRF_TOKEN 後，get-conversation-metadata 成功。
created: 2026-10-31
updated: 2026-10-31
tags:
  - antigravity
  - agentapi
  - csrf
---

# Antigravity IDE agentapi 喚醒既有對話

IDE 裡正在跑的對話要從外面戳醒，用 `language_server.exe agentapi`，不是 `agy` CLI。`agy --conversation` 對應的是 CLI 對話，和 `~/.gemini/antigravity/brain/<id>` 不是同一套。

`agentapi` 沒帶環境變數時會回 `ANTIGRAVITY_LS_ADDRESS is not set`。位址與 token 每次啟動都變，不要寫進檔案。

1. 從正在跑的 `language_server.exe` 命令列讀 `--csrf_token`。不要印出 token。
2. 用 `netstat` 找出該 PID 正在聽的 `127.0.0.1` port。通常有兩個。
3. 被對方直接關掉連線的那個 port 不是 agentapi。回 `missing CSRF token` 的那個才是。
4. 設定後再呼叫：

```text
ANTIGRAVITY_LS_ADDRESS=<那個 port 的 127.0.0.1:port>
ANTIGRAVITY_CSRF_TOKEN=<命令列上的 csrf token>
agentapi get-conversation-metadata <conversation-id>
agentapi send-message <conversation-id> <content>
```

只有 `ANTIGRAVITY_CSRF_TOKEN` 這個名字有效。`CSRF_TOKEN`、`ANTIGRAVITY_LS_CSRF_TOKEN` 仍會被當成沒帶 token。

`send-message` 的收件者用 brain 對話 id。這種訊息會以系統訊息進同一段對話，用來在代理人自己結束回合、五分鐘巡檢沒有新 planner step 時叫它繼續看 Sixty Four。背景中的 keeper 或 cron task 仍顯示 RUNNING，不代表 planner 還在做事。
