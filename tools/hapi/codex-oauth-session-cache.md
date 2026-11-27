---
title: HAPI Codex session 的 OAuth 快取
scope: tool
tool: hapi
tags: [hapi, codex, oauth]
status: active
created: 2026-08-30
updated: 2026-11-27
---

# HAPI Codex session 的 OAuth 快取

`codex login` 更新 `~/.codex` 憑證後，已啟動的 HAPI Codex `app-server` 不會自動重讀。此時 `codex login status` 可顯示已登入，但既有 Task 仍可能在 `/v1/responses` 收到 401（缺少 Authorization header 或 token 已撤銷）。

處理方式：確認受影響的精確 HAPI session，停止該 session 的 Codex parent process，再 resume 同一 session。用 session 實際產生 agent 回應驗證；只看 `codex login status` 不足以證明既有 app-server 已更新認證。

## 2026-11-27 refresh 已輪替，但 access JWT 尚未過期

Swever 上兩個新 Codex session 一啟動就收到 `token_expired`，接著是 refresh credential 已被用過。Auth 檔的 access JWT `exp` 仍在隔天，所以不能只看本機到期時間。同一台的 Pi `openai-codex` 在啟動時剛換到同一 ChatGPT 帳號的新 access 與 refresh。把這兩筆與 account id 寫回 Codex auth 檔後，`codex login status` 為 ChatGPT，`GET /backend-api/models` 回 200。

Codex CLI 會拒絕缺少 id token 欄位的 auth 檔（`invalid credential JSON`）。只能替換 access、refresh 與 account id，並保留原 id token 欄位。不要在寫入前記錄或印出 token。

已啟動的 app-server 不會重讀檔案。對 app-server 子程序送 TERM，HAPI parent 會跟著退出（runner 記 `app-server exited`），session 變 inactive。用同一筆 hub session resume，不要另開一筆。2026-11-27 resume 後，該 Codex session 回覆了確認句，沒有再報 refresh 失敗。

Pi 與 Codex 若同時握有同一組 refresh，下一次輪替會再讓其中一邊失效。這次只讓 Codex 載入 Pi 當時的有效組合，沒有再主動刷新。

重開 Codex peer 前，可先用一次真實請求確認 CLI 認證可用：`cd /tmp && codex exec --skip-git-repo-check "Reply with exactly: ok"`，回 `ok` 才算通過（約 1.6 萬 tokens）。啟動即失敗的 peer 直接用 hapi-session-control 的 `archive-peer.py` 歸檔，修好認證後再開新的。
