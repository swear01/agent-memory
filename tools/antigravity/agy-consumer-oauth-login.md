---
title: Antigravity CLI (agy) consumer OAuth 登入：hosted callback、等待上限與手動貼 code
scope: tools/antigravity
tool: Antigravity CLI (agy)
status: active
confidence: high
evidence: >-
  以 agy 1.2.1 在本機實測：攔截實際 authorization URL、比較無效 code 與 PKCE
  不符時的 Google 回應差異、量測 print 模式與 TUI 的等待上限、以 strace 確認
  token 落地路徑。
created: 2026-09-11
updated: 2026-11-28
tags:
  - antigravity
  - agy
  - oauth
  - pkce
  - headless
---

## OAuth 登入失敗

以下行為以 agy 1.2.1 實測。執行 `agy` 選 **Google OAuth**。該版使用 Authorization Code + PKCE S256 和 hosted callback `https://antigravity.google/oauth-callback`，要將該輪瀏覽器顯示的 code 貼回同一個 TUI。沒有本機 callback listener，也沒有 `agy auth login`。

| 症狀 | 下一步 |
| --- | --- |
| `invalid_grant: Invalid code verifier.` | 已觀察到 code 與該輪 PKCE verifier 不符。關閉本次舊 OAuth 分頁，重開一輪並只使用完整的新 URL 和新 code。重試仍失敗時核對登入入口和帳號，不憑錯誤猜有其他程式搶發 code。 |
| `invalid_grant: Bad Request` | 該測試中無效 code 會得到此訊息。先查是否截斷、誤貼或過期。 |
| `Error: authentication timed out.` | `agy -p` 當時約 60 秒超時。手動登入走互動 TUI，實測 190 秒後仍可等待貼上。 |
| TUI 停在 `Signing in...` | 用 PTY 驅動時送 CR（`\r`）提交，print 模式則讀 LF。窄終端可能截斷長 URL，要從同一輪取完整單行。 |

## 驗證登入

- 無 keyring 時當時會 fallback 到 `~/.gemini/antigravity-cli/antigravity-oauth-token`。該版未發現 `GEMINI_FORCE_FILE_STORAGE`。只用檔案 metadata 查是否新寫入，不讀出憑證。
- 啟動瞬間出現 `You are not logged into Antigravity` 可能是背景 worker 早於憑證載入。用 `agy -p "reply only with OK"` 驗同帳號的實際推論。
- Oracle 已有 `agy-login` helper 使用寬 PTY，預設走 TUI。由其本輪完整 URL 進入瀏覽器。保留同一登入程序與 verifier；核對 token 檔變更後仍要驗推論。開始前只結束自己的舊登入嘗試。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/tools/antigravity/agy-consumer-oauth-login.md)
