---
title: "Prompt 很短仍要檢查實際送出的 request 欄位"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: aaacb80b9b84f60f2c4e1752418eb3ed3c7fd3554c702230ca989f80270f552e
---

# Prompt 很短仍要檢查實際送出的 request 欄位

歷史助手估計 prompt 約千 token，API 卻回報遠大用量；實際欄位清單顯示 property_desc 超過兩百萬字元，助手追到 prompt builder 讀入該欄位。

檢查實際序列化 request 的欄位與大小，對不需要的上游大型內容採明確省略或有界摘要，並保留截斷狀態與語意限制。選中來源沒有草稿所稱的 HTTP 413 輸出，也沒有修正後重跑，不採用這兩項結論。
