---
title: Pi 的 OpenCode Go 憑證會蓋掉本機 gateway
scope: tools/pi
status: active
created: 2026-11-27
updated: 2026-11-27
tags:
  - pi
  - opencode-go
  - deepseek-gateway
---

# Pi 的 OpenCode Go 憑證會蓋掉本機 gateway

Pi 把 `opencode-go/deepseek-flash` 打到本機 gateway 時，若 agent auth 檔裡還存著 OpenCode Go 的 api key，那個儲存值會蓋掉 `models.json` 的 `local-gateway`。Gateway 只接受 marker，其他值回 HTTP 401，正文是 `Invalid API key`。這不是 upstream key 失效，也不該因此去另一台機器要新 key。

## 2026-11-27 swever 已驗證

- `models.json` 的 `opencode-go.baseUrl` 是 loopback `:35001/v1`，`apiKey` 是公開 marker `local-gateway`。
- 同一 provider 的 auth 檔若有 `type: api_key`，Pi 的 provider composer 走 stored credential，不再用 `models.json` 的 key。
- 拿 auth 檔裡那把 OpenCode Go key 打 gateway 的 chat completions，得到 401 與上述正文。拿 `local-gateway` 打同一路徑得到 200，served model 為 `deepseek/deepseek-v4.1-flash`。
- 那把 OpenCode Go key 直接向 OpenCode 的 models 端點查詢仍是 200。`deepseek-flash` 這個 id 不在 OpenCode Go 目錄裡，直連會是 400，不是這次的 401。
- 修法：從 Pi auth 檔移除 `opencode-go` 這一筆，讓 `models.json` 的 marker 生效。Upstream key 繼續只放在 gateway 讀的 secrets 檔。已啟動的 Pi 會把 auth 檔讀進記憶體，改檔後要重開該 session 才會生效。
- 重開後對該 Pi session 送一句確認，它回了正確的 401 原因，沒有再報 Invalid API key。
