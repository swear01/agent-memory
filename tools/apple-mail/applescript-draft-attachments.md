---
title: NTU M365 工作信箱附檔：Apple Mail 草稿與寄件驗證
scope: tools/apple-mail
status: active
confidence: high
created: 2026-10-20
updated: 2026-11-28
tags: [apple-mail, applescript, attachments, ntu-m365]
---

依全域 AGENTS.md，NTU M365 工作信箱平時用 Outlook connector，需附檔時用 Apple Mail。Gmail 與 NTU 學生信箱使用 [Himalaya](../himalaya/himalaya-ortie-setup.md)，學生 SMTP 為 `himalaya -a ntu`。寄出前列出收件人、主旨與附件，並核對實際寄件帳號。

## 草稿

Apple Mail 可直接附上本機檔案。以 `osascript - "<寄件帳號>" "<收件地址>" "<主旨>" "<正文>" "<附件完整路徑>"` 執行草稿腳本，讓 `on run argv` 處理參數和中文路徑。

```applescript
on run argv
  set {senderAddress, recipientAddress, subjectText, bodyText, attachmentPath} to argv
  tell application "Mail"
    set m to make new outgoing message with properties {subject:subjectText, content:bodyText, sender:senderAddress, visible:true}
    tell m
      make new to recipient with properties {address:recipientAddress}
      tell content
        make new attachment with properties {file name:(POSIX file attachmentPath)} at after the last paragraph
      end tell
    end tell
    save m
  end tell
end run
```

## 驗證與回復

- 草稿的 `count of attachments of outgoing message` 曾回 0，讀 `mail attachments` 時也可能報 `-1728`。用草稿視窗核對附件圖示、檔名及郵件大小。寄出後對實際寄件帳號的 Sent mailbox 查該封信，`message` 物件上的 `count of mail attachments` 可作讀回。
- 寄送前核對同一份草稿的 From、To、Subject 和附件。若透過 AppleScript 選取草稿，只有主旨相同不足以確認目標；需核對欄位和唯一性後才處理。
- `close m saving no` 不會移除已儲存的草稿。需要移除時只處理已核對的目標草稿，並重新整理後讀回，因為同步可能略有延遲。

[原始實測與歷史做法](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/apple-mail/applescript-draft-attachments.md)
