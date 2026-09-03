---
title: 個人憑證/密碼/keys 的統一存放位置（Google Drive passcode 資料夾）
scope: machine
machine: swairM5
status: active
created: 2026-09-03
updated: 2026-09-03
---

# 憑證統一存放位置

所有帳號密碼、API keys、recovery codes 已收錄到單一位置，**雲端 Drive 端是唯一存放點**（本機不保留獨立 `~/credentials` 資料夾，無 age 加密、無 cred 指令）：

- 本機路徑：`<remote-home>/Library/CloudStorage/GoogleDrive-<drive-account>/我的雲端硬碟/document/passcode/`（Drive 自動同步）
- 結構：
  - `passwords.txt` — 全部帳號/密碼/PIN，`service: value` 格式（含實驗室 server swear01/swear02、snps、jizhong、eduroam、RustDesk、3060 SN、自然人憑證用戶代碼 `\\*S:"\n."`、VC 數位憑證驗證代碼等）
  - `api-keys.txt` — 全部 API keys/tokens，`service: value` 格式（4 個 GitHub PAT、5 個 Google OAuth、Discord/Telegram bot、Blackblaze bucket、Groq、OpenRouter、DeepSeek、OpenAI、ElevenLabs、HackMD、Zenodo 等）
  - `recovery-codes/` — 每 app 一個 txt，一行一個 code（microsoft、steam、epic、nintendo、slack、blackblaze、cloudflare、discord 系、github、parsec、pixiv、arc 等 21 檔）
  - 同資料夾另有舊檔（`SwearPasswords.kdbx`、`Brave Passwords.csv`、`SwearOVPN.ovpn`）

## 規則

- 以後要查密碼/key/recovery code：直接讀上述檔案，不要再問用戶或翻舊記憶。
- 新增憑證時寫進對應檔案（密碼→`passwords.txt`、key→`api-keys.txt`、recovery code→`recovery-codes/<app>.txt`），維持既有格式。
- 注意：`recovery-codes/pixiv.txt` 原始檔曾含 CRLF，處理舊檔時先 `tr -d '\r'`。
- 本機 FileVault 已啟用；檔案權限 600、資料夾 700。