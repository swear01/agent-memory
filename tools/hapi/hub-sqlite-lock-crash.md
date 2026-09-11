---
title: 直接開 hub SQLite 會讓 HAPI hub 崩潰（database is locked）
scope: tools/hapi
project: hapi
tool: hapi
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-11
tags:
  - hapi
  - hub
  - sqlite
  - readonly
  - incident
---

# 現象

2026-09-11 在 mazu 上用 `bun:sqlite` 以 `readonly: true` 直接開
`/var/tmp/hapi-hub/hapi.db` 讀 sessions/messages 之後，hub service 兩次以
`SQLiteError: database is locked`（exit code 1）結束，間隔分別約 12 分鐘與 2 小時；
systemd 依 `Restart=always` 重新拉起。崩潰時正對應到直接讀 DB 的時間點。

# 教訓

- 查 hub 資料要用 HTTP API：`POST /api/auth` 以 `CLI_API_TOKEN` 換 JWT，再打
  `/api/machines`、`/api/sessions`、`/api/machines/:id/pi-models`。唯讀 API 已經涵蓋大部分
  診斷需求，且不影響 hub process。
- `readonly: true` 不代表安全：WAL 模式下 reader 仍需要 `-shm` 檔與共享鎖；hub 端若遇到
  無法取得的鎖，會把它當致命錯誤往上拋而不是重試。
- 若真的非得看 DB，只能在 hub 停止時讀，或先做 `VACUUM INTO` 複本再讀複本，
  絕不在 hub 執行中直接開正式檔。
- 已知 `messages` 表把 payload 以 zstd blob 儲存；用 `Bun.zstdDecompressSync` 才能解出內容，
  `gunzip`/`inflate`/`brotli` 都會失敗（header `28 b5 2f fd`）。這也是為什麼用 API 比讀 DB 省事。
- `hapi inspect-peer <id>` 對「自己這個 runner 上、由 runner 啟動」的 session 可能回
  `Session not found`；同一台機器上另一個用戶端（peer）建立的 session 則可正常 inspect。
