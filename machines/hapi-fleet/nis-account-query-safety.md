---
title: NIS 帳號查詢不得輸出原始 passwd record
scope: machines/hapi-fleet
tags: [nis, nss, credentials, account-audit]
status: active
created: 2026-09-04
updated: 2026-09-04
---

# NIS 帳號查詢不得輸出原始 passwd record

2026-09-04 在 Mazu 實測，`getent passwd <NIS-user>` 的 password 欄位可能回傳
完整 password hash，而不是一般本機 shadow 設定中的 `x`。因此不得把原始 record
輸出到 terminal、agent transcript、報告或 memory。

只需確認帳號存在或 UID 時，使用 `id -u <user>`。若確實需要 home／shell 等欄位，
必須在輸出前於主機端依冒號分欄並排除第二欄；不要先把整筆內容帶回 agent。
任何已進入 transcript 的 password hash 都視為 credential exposure：不重複、不保存，
並建議管理者更換受影響帳號的密碼。

帳號存在只適合作為資料清理的保守 HOLD gate，不等於近期登入證據；登入活動仍需
由中央 roster 或具足夠 retention 的跨主機 audit log 證明。
