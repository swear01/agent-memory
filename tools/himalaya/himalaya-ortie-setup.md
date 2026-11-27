---
title: Himalaya 2.x + ortie 作為 AI 寄信工具（Gmail、NTU 學生信箱、NTU M365）
scope: tools/himalaya
tool: himalaya
status: active
created: 2026-10-20
updated: 2026-11-28
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
- NTU 學生信箱只有 POP3，himalaya 只有 IMAP 等 backend、沒有 POP3，所以 `ntu` 帳號只能寄。Gmail 之後會用 POP3 把校園信箱拉進標有校園帳號的標籤，日常閱讀仍可用 `-a gmail`，但不能拿它來確認剛剛寄出的信（見下方「以校園 SMTP 寄信與立刻核對」）。
- `permissions.ask` 規則加好後兩次測試寄信都成功，但當時沒確認使用者端是否真的跳出確認視窗；之後若發現沒跳，檢查 pattern 是否匹配 `himalaya … --send`。
- auto mode 分類器會擋 agent 自己跑 `ortie auth get` 與讀 Keychain metadata；授權步驟要使用者在自己的 terminal 執行。

## 以校園 SMTP 寄信與立刻核對（2026-11-28）

- **Gmail API 不會改走校園 SMTP**。Gmail 的 `create_draft` / `send_draft` / `send_email`，以及 DraftExternalMessage，都會忽略 send-as 裡的 `smtpMsa`，改由 `gmailapi.google.com` 寄出，Message-ID 網域是 `mail.gmail.com`。只有 Gmail 網頁或官方 App 才會使用 `smtpMsa`。若要從校園帳號經臺大伺服器寄出，不要走 Gmail API。
- **在 SwairM5 可靠的寄法**是 `himalaya -a ntu`，SMTP 為 `smtps.ntu.edu.tw:465`。要明確指定 `--from` 為校園地址，並加 `--no-save`（`ntu` 帳號只有 SMTP，himalaya 無法把寄件備份寫回這顆只能 POP3 的信箱）。附件用 `--attach`。himalaya 2.2.1 沒有 `template` 子命令，也不再使用 `mml`。
- 要回覆既有信件並保留串接時，用 Python `email.message.EmailMessage` 寫好含 `In-Reply-To`、`References` 的 `.eml`，再交給 `himalaya -a ntu` 寄出，不要指望 himalaya 自己組這些標頭。
- **不要等 Gmail 出現密件副本**。Gmail 是用 POP3（`msa.ntu.edu.tw:995`）定時去拉校園信箱，間隔可能拉長到數小時，不能當寄出證明。密件副本仍抄給同一顆校園信箱即可。
- **立刻核對**的做法：用唯讀 `POP3_SSL` 連 `msa.ntu.edu.tw:995`，只做 `STAT` 與 `TOP`，不要 `DELE`。密碼從 macOS Keychain 讀（`-s himalaya -a ntu`），不要把密碼寫進指令歷史或筆記。確認 Message-ID 網域是臺大郵件閘道，而不是 `mail.gmail.com`，才能證明真的走 `smtps.ntu.edu.tw`。

相關：[[applescript-draft-attachments]]、[[ntu-student-mail-client-setup]]、[[gmail-attachment-raw-extract]]
