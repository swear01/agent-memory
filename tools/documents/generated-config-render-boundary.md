---
title: "文件 metadata 必須在中間產物與輸出中生效"
scope: "tools/documents"
status: active
updated: 2026-09-15
evidence_digest: 1dc6ce6c291b3f0e5baf63322fe12237c89c8f09db62a50f20ae5ebd648ac443
---

# 文件 metadata 必須在中間產物與輸出中生效

歷史文件轉換記錄中，Agent 發現 Pandoc 的 margin metadata 沒有進入產出的 Typst conf，PDF 因此仍使用 template 的預設邊界。後續訊息說已把設定寫進 Typst，接著才要編譯。

調整文件版面時，檢查實際產出的中間設定，再編譯並渲染查看最終頁面。命令接受 metadata、來源檔含有新值，或編譯成功，都不足以單獨證明頁面邊界與字型已生效。

來源支持當次 pipeline 的設定未傳遞；不代表所有 Pandoc template 都忽略 margin，也不證明直接修改後的 PDF 已渲染驗證。
