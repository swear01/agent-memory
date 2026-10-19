---
title: 從 Apple Mail 本機 emlx 搜尋信件
scope: tools/apple-mail
status: active
confidence: high
created: 2026-10-19
updated: 2026-10-19
tags:
  - apple-mail
  - emlx
  - gmail
  - ntu-cool
---

# 從 Apple Mail 本機 emlx 搜尋信件

Gmail／Drive connector 沒授權時，macOS 的 Apple Mail 本機快取可以直接搜，不需要開 app 也不需要 computer use。2026-10-19 用這個方法找到 NTU COOL 轉寄的課程公告。

- 位置：`~/Library/Mail/V10/<account-uuid>/<mailbox>.mbox/.../Messages/*.emlx`；Gmail 帳號會有 `[Gmail].mbox/全部郵件.mbox`，另一個帳號的 `INBOX.mbox` 可能存著同一封信的副本。
- 格式：第一行是 byte 數，接著是那麼多 bytes 的 RFC 822 原文，後面是 plist。用 `raw.split(b'\n',1)` 取長度，再用 `email.message_from_bytes(rest[:n], policy=policy.default)` 解析。
- 直接 `grep` 原始檔不可靠：內文多半是 base64 或 quoted-printable，中文關鍵字搜不到，短的英文字（如 `ECC`）又會誤中 base64。要先解析、取 `text/plain`／`text/html` 解碼後的內容，再比對主旨、附件檔名與內文。
- NTU COOL 收件匣的訊息會以「<老師> (<課名>)剛剛在 NTU COOL 中傳送了訊息給您」為主旨轉寄，內文包含公告全文與 `cool.ntu.edu.tw/conversations/<id>` 連結。
