---
title: "聽寫 Send 必須讓滑鼠與觸控也可達"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 93aabecd1f5d2d81538905e3b31f71d200516b3e73266717f862cb13c920afda
---

# 聽寫 Send 必須讓滑鼠與觸控也可達

歷史 review 指出 connected-dictation 只有 Stop 按鈕，stopAndSend 只能由 Enter 觸發，因此滑鼠與觸控缺少送出路徑。助手提出在 Stop 旁增加明確 Send。

依已定 UX 保留 stop 與 send 的區別，同時讓必要操作可由支援的輸入方式完成。不能為補按鈕就把 Stop 改成送出，反轉既有行為。來源止於有效 finding 與修法提案，沒有實作或驗證結果。
