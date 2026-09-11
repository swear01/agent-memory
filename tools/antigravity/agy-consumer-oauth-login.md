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
updated: 2026-09-11
tags:
  - antigravity
  - agy
  - oauth
  - pkce
  - headless
---

## 症狀

`agy` 在互動登入時出現：

```text
Got an error: token exchange failed: oauth2: "invalid_grant" "Invalid code verifier."
```

## 架構事實（決定所有失敗模式）

`agy` 的 consumer 登入是 Authorization Code + PKCE（`S256`），但 **redirect_uri 是 hosted callback，不是 loopback**：

```text
auth  : https://accounts.google.com/o/oauth2/auth
token : https://oauth2.googleapis.com/token
redirect_uri : https://antigravity.google/oauth-callback
scope : cloud-platform, userinfo.email, userinfo.profile, cclog, experimentsandconfigs, aicode, openid
```

因此**沒有本機 listener 可以自動接收 code**，CLI 只能把 code 印在瀏覽器上請使用者手動複製貼回。每次登入都產生新的 `code_challenge` / `code_verifier`，且 code 為單次使用，所以下列行為一定會壞：

- 貼上「上一次」嘗試（或殘留分頁）產生的 code；
- 從被終端硬換行的 URL 複製（URL 長約 704 字元，窄終端會截斷）；
- 只是單純超時。

`agy` 沒有 `auth login` 子命令（未知子命令只印 usage）；登入入口就是直接跑 `agy` 後選 `Google OAuth`。

## 以 Google 回應區分兩種失敗（實用診斷）

對同一台機器的 `agy` 分別餵入不同 code：

| 輸入 | Google 回應 |
| --- | --- |
| 不存在的 code | `oauth2: "invalid_grant" "Bad Request"` |
| 真實但 PKCE 不符的 code | `oauth2: "invalid_grant" "Invalid code verifier."` |

`Bad Request` 代表 Google 完全不認識該 code（打錯／被截斷）；`Invalid code verifier.` 代表 code 真實存在但 challenge 對不上。看到後者就不要再懷疑複製品質，一定是「code 來自另一次登入嘗試」。

## 等待上限：print 模式有硬性 60 秒，TUI 沒有

| 模式 | 等待上限 | 逾時訊息 |
| --- | --- | --- |
| `agy -p` / `--print` | **硬性 60 秒**（實測 60.72s） | `Error: authentication timed out.` |
| 互動 TUI（直接跑 `agy`） | **無此上限**（實測 >190s 仍停在 paste 畫面） | 無 |

60 秒要完成 Google 登入＋同意頁＋複製貼回過於緊迫，這是反覆失敗最常見的環境成因。**需要手動登入時一律用互動 TUI，不要用 `-p`。**

## TUI 是 raw mode：Enter 必須送 `\r`

互動 TUI 走 bubbletea raw mode，Enter 是 CR；`-p` 模式讀一般行，接受 LF。用 pty 自動化時若對 TUI 送 `\n`，code 只會被「打進」輸入框而不會送出，畫面停在 `Signing in...`，形成靜默失敗。正確做法是 TUI 送 `\r`、print 模式送 `\n`。

## Token 落地位置與 keyring fallback

以 `strace` 確認：

```text
~/.gemini/antigravity-cli/antigravity-oauth-token
```

JSON 結構為 `auth_method`、`id_token`、`token`（內含 access token、expiry、refresh token、token type）。`auth_method: consumer` 表示個人 Google 帳號。

headless／無 keyring 時會 fallback 到檔案，log 可見：

```text
composite_token_storage.go:419] Failed to load stored token from keyring, falling back to file:
  The name org.freedesktop.secrets was not provided by any .service files
```

**`GEMINI_FORCE_FILE_STORAGE=true` 對 `agy` 無效**——`strings` 掃描二進位完全沒有這個變數名。網路上常見的這個建議是誤植（來自其他工具）。

## 啟動時大量 `not logged in` 是無害競態

登入成功後，log 仍可能出現數十筆：

```text
error getting token source: You are not logged into Antigravity.
```

若這些全部集中在同一秒的數毫秒內（背景 worker 早於憑證載入），屬於啟動競態，不是登入失敗。判斷真實狀態要用功能驗證：

```bash
agy -p "reply only with OK"
```

## 可重用的登入輔助腳本

本機 `<remote-home>/agy-login`（Python，stdlib）封裝了上述所有修正：

- 以 pty 驅動 `agy`，SSH／headless（`TERM=dumb`）也能走手動貼 code 路徑；
- 配置**極寬 pty（2000 欄）**使 URL 不換行，並用 schema 驗證（`access_type/client_id/code_challenge/S256/state` 等）確保寫出的 URL 完整；截斷者拒收並警告；
- 單次執行只產生一個 PKCE verifier，絕不偷偷重啟流程；
- code 可來自終端或檔案，stdin 關閉也不中止；
- 以 token 檔 mtime 變化判定真正登入成功，避免把舊憑證誤判為本次成功；
- 預設用 TUI（等待無上限），`--print` 走 60 秒模式，`--keep` 登入後留在 TUI。

```bash
python3 <remote-home>/agy-login            # 建議：TUI
python3 <remote-home>/agy-login --print    # 60s 硬限
```

操作要點：URL 會寫到 `/tmp/agy-login-url.txt`（完整單行），用**全新分頁**開啟，完成後複製最新 code 貼回；若 stdin 不可互動，另一 shell 執行 `echo '<code>' > /tmp/agy-login-code.txt`。

## 復發時的排查順序

1. `pkill -f '^agy$'`，關閉所有舊的 antigravity/OAuth 分頁（避免舊 code 污染）。
2. `python3 <remote-home>/agy-login`（TUI，等待無上限）。
3. 從 `/tmp/agy-login-url.txt` 複製 URL 到全新分頁，60 秒內貼回 code。
4. 驗證：token 檔存在且 `agy -p "reply only with OK"` 回 `OK`。
5. 若仍為 `Invalid code verifier.`，代表有其他工具／分頁共用同一 OAuth client 搶發 code；改用另一種登入路徑（TUI 第二選項 `Use a Google Cloud project`，或 Gemini API key／ADC）。
