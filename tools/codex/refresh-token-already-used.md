---
title: Codex「refresh token was already used」時 login status 仍顯示已登入
scope: tools/codex
status: verified
updated: 2026-11-27
tags: [codex, auth, chatgpt, hapi]
---

2026-11-25 在 swever 用 HAPI `spawn_peer` 開 Codex session，第一輪就失敗：`Codex thread entered systemError`，接著 `Your access token could not be refreshed because your refresh token was already used. Please log out and sign in again.` session 沒做任何事。

- 同時 `codex login status` 仍印 `Logged in using ChatGPT`，不能拿它判斷登入可用。
- 用真實請求驗證：`cd /tmp && codex exec --skip-git-repo-check "Reply with exactly: ok"`，回 `ok` 才算可用（約 1.6 萬 tokens）。
- 修法需要使用者在該機器重新登入：`codex logout && codex login`（瀏覽器授權，agent 無法代做）。使用者重新登入後，同一台機器重開 Codex peer 即正常。
- 失敗的 peer 直接用 hapi-session-control 的 `archive-peer.py` 歸檔，不要對同一個壞掉的 session 重送任務。
- 原因推測：同一份 `auth.json` 的 refresh token 已被另一個程序或共用 home 的機器先刷新（未驗證）。

相關：[ChatGPT 訂閱登入設定](chatgpt-subscription-auth.md)。
