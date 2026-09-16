---
title: "Publickey 成功後仍可能在 PAM account 階段退出"
scope: tools/ssh
status: active
updated: 2026-09-16
evidence_digest: 98ab6653fbde95a51d2aedca28aa9ad2a6c47777fb0495f5b20df55cfb37718c
---

# Publickey 成功後仍可能在 PAM account 階段退出

歷史助手回報 SSH 已接受 key，但 PAM 要求更新到期密碼，非互動連線沒有 TTY 而退出；因此不能把整段失敗當成 key 不可用。

依認證、account policy 與 session 啟動階段讀取 trace，再走使用者或管理員正常的帳號處理流程。不要因 key 已通過就繞過到期政策。來源沒有實際重設或重新登入結果；同段另有 skill 放置問題，未合併為此根因。
