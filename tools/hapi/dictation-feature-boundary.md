---
title: "查聽寫問題先辨識 Dictation 的實際路徑"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: b32621b38304c8379e7978bd6517e9ffed8d9ee8e573d2e3f14794cb279f64f0
---

# 查聽寫問題先辨識 Dictation 的實際路徑

使用者詢問語音語言設定，歷史助手承認先誤查完整語音助理，改讀 Dictation 的 hook 與 route；隨後列出不同 provider 的語言碼處理與輸出直接插入方式。

先確認使用者選的功能、provider、標準或即時模式，再追傳送欄位與回傳文字。不要拿另一套語音功能解釋此問題，也不要把語言提示直接當強制字體保證。來源只有助手閱讀回報，沒有 wire capture；列出的 API 欄位、地域碼支援與繁簡成因仍需獨立查證，本筆不採用為確定規格。
