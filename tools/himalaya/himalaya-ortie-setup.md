---
title: Himalaya 2.x + ortie 作為 AI 寄信工具（Gmail、NTU 學生信箱、NTU M365）
scope: tools/himalaya
tool: himalaya
status: active
created: 2026-10-20
updated: 2026-10-20
tags:
  - himalaya
  - ortie
  - oauth
  - gmail
  - m365
  - ntu
---

# Himalaya 2.x + ortie 作為 AI 寄信工具

2026-10-20 在 macOS（SwairM5）設定完成並實測雙向寄出帶 PDF 附件的信。取代 Gmail connector（附件要 base64 進參數）與 AppleScript（附件無法驗證）的寄信路徑。

## 安裝與設定位置

- `brew install himalaya`（2.2.1，預設已含 imap/smtp/gmail/msgraph backend）。
- 2.x 不再內建 OAuth，交給 `ortie`；Homebrew 沒有，`cargo install --locked --root ~/.local ortie`（約 5 分鐘）。
- 設定：`~/.config/himalaya/config.toml`、`~/.config/ortie/config.toml`。帳號 `gmail`（預設，IMAP/SMTP XOAUTH2）、`ntu`（只有 SMTP）。
- Token 存 macOS Keychain：`storage.read.command = ["security","find-generic-password","-s","ortie","-a","<acct>","-w"]`、`storage.write.command = 'security add-generic-password -U -s ortie -a <acct> -w "$(cat)"'`。NTU 密碼存 `-s himalaya -a ntu`。
- 附件直接用 `himalaya -a <acct> message compose -t … -s … --body … --attach <file> --send`，不需要 `mml`。驗證用 `envelope list -m sent` 與 `attachment list -m sent <id>`。
- Claude Code `~/.claude/settings.json` 加 `permissions.ask: ["Bash(himalaya *send*)"]`，寄信一律跳確認。

## 陷阱

- **ortie redirect 不能用 `http://localhost`**：等於 port 80，報 `Permission denied (os error 13)`。Gmail 改 `endpoints.redirection = "http://localhost:8765"`（Google loopback 允許任意 port）。
- **Gmail IMAP 資料夾名稱跟著介面語言**：中文帳號是 `[Gmail]/寄件備份`、`[Gmail]/草稿`、`[Gmail]/垃圾桶`，要設 `mailbox.alias.*`，否則 `-m sent` 找不到。
- **NTU M365（`<id>@m365.ntu.edu.tw`）擋第三方 OAuth**：用 Thunderbird 公開 client（`9e5f94bc-…`，IMAP/SMTP scopes）出現 `Need admin approval`；自建 Azure app 也會遇到同樣的租戶同意政策。不要用 Microsoft 第一方 client ID 繞過。工作信箱改走 claude.ai Outlook connector（不能附件），附件用 Apple Mail；若計資中心核准，再把 `work` 帳號（`outlook.office365.com` / `smtp://smtp.office365.com:587` + STARTTLS）加回。
- NTU 學生信箱只有 POP3，himalaya 只有 IMAP 等 backend、沒有 POP3，所以 `ntu` 帳號只能寄。學生信箱的信會進 Gmail（Gmail 有 `<student-id>@ntu.edu.tw` 標籤，寄到學生信箱的測試信出現在 Gmail 收件匣），讀信用 `-a gmail`。
- `permissions.ask` 規則加好後兩次測試寄信都成功，但當時沒確認使用者端是否真的跳出確認視窗；之後若發現沒跳，檢查 pattern 是否匹配 `himalaya … --send`。
- auto mode 分類器會擋 agent 自己跑 `ortie auth get` 與讀 Keychain metadata；授權步驟要使用者在自己的 terminal 執行。

相關：[[applescript-draft-attachments]]、[[ntu-student-mail-client-setup]]、[[gmail-attachment-raw-extract]]
