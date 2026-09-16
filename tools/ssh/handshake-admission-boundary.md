---
title: "SSH 握手拒絕不能直接推論主機死亡"
scope: tools/ssh
status: active
updated: 2026-09-16
evidence_digest: 8fcb16d9af572f9602c7f1a37a930f0ee9f62e5bab0fa5ff42c9baa0d8b0d532
---

# SSH 握手拒絕不能直接推論主機死亡

歷史助手先把 SSH timeout 描述成主機掛掉，之後 verbose 輸出顯示 Exceeded MaxStartups，表示当時有 SSH 服務回覆並拒絕握手。

將 TCP 可達、SSH 握手、認證與登入後工作分開診斷。明確的 admission 錯誤不能當成主機已關機證據；保留有界重試與其他已知狀態線索，不以無限重連加重問題。來源未確認大量半開連線或負載是根因，也沒有恢復結果。
