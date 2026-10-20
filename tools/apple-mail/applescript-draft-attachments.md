---
title: 用 AppleScript 在 Apple Mail 建立帶附件的草稿與寄出
scope: tools/apple-mail
status: active
confidence: high
created: 2026-10-20
updated: 2026-10-20
tags:
  - apple-mail
  - applescript
  - osascript
  - attachments
---

# 用 AppleScript 在 Apple Mail 建立帶附件的草稿與寄出

Gmail connector 的 `create_draft` 要把附件 base64 塞進參數（幾百 KB 的 PDF 會吃掉大量 token），Outlook connector 的草稿不支援附件。macOS 上改用 `osascript` 讓 Apple Mail 直接讀本機檔案，零 token 成本，也能指定寄件帳號。2026-10-20 用此方法寄出 7 張實驗室採購發票 PDF。

> 2026-10-20 起 Gmail 與 NTU 學生信箱改用 himalaya 寄信（見 [[himalaya-ortie-setup]]）；這個方法只留給 NTU M365 工作信箱需要附件時。

## 做法

```applescript
tell application "Mail"
  set m to make new outgoing message with properties {subject:"...", content:b, sender:"<account-address>", visible:true}
  tell m
    make new to recipient with properties {address:"<recipient>"}
    tell content
      make new attachment with properties {file name:(POSIX file "<path>.pdf")} at after the last paragraph
    end tell
  end tell
  delay 2
  save m
end tell
```

- 檔案路徑透過 `osascript - "$DIR" <<'EOF' ... on run argv` 傳入，避免中文路徑在 shell 引號裡出錯；Google Drive for desktop 的 `CloudStorage/.../我的雲端硬碟` 路徑可直接用。
- `sender` 要填 Mail 帳號的完整地址（`get email addresses of every account` 可查）。

## 陷阱

- `count of attachments of outgoing message` 與 `mail attachments of outgoing message` 對草稿永遠回 0 或報錯 `-1728`，**不能拿來驗證**。改用 computer-use `app_screenshot` 看草稿視窗底部的附件圖示與「郵件大小」。
- `close m saving no` 只關掉視窗；已 `save` 過的草稿仍留在 `drafts mailbox`，要另外 `delete (every message of drafts mailbox whose subject ...)`。刪除後立即查詢仍會列出，約 3 秒後才同步消失。
- 寄出用 `send item 1 of (every outgoing message whose subject is "...")`；先確認數量為 1 再送。寄出後驗證要掃 `every mailbox of every account`（NTU POP 帳號的寄件備份叫 `Sent Messages`），`message` 物件上的 `count of mail attachments` 才是可信的附件數。
