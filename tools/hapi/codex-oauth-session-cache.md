---
title: HAPI Codex session 的 OAuth 快取
scope: tool
tool: hapi
tags: [hapi, codex, oauth]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# HAPI Codex session 的 OAuth 快取

`codex login` 更新 `~/.codex` 憑證後，已啟動的 HAPI Codex `app-server` 不會自動重讀。此時 `codex login status` 可顯示已登入，但既有 Task 仍可能在 `/v1/responses` 收到 401（缺少 Authorization header 或 token 已撤銷）。

處理方式：確認受影響的精確 HAPI session，停止該 session 的 Codex parent process，再 resume 同一 session。用 session 實際產生 agent 回應驗證；只看 `codex login status` 不足以證明既有 app-server 已更新認證。
