---
title: "單一依賴解析端點故障先查等價來源"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: 8a44aae7e95f6fe055b486f873d0cc9d759a653b132a28ec9c1c01b53c359edd
---

# 單一依賴解析端點故障先查等價來源

歷史助手在 EMI resolver 兩次遇到上游 HTTP 502 後，只提供等待或略過驗證兩種選擇；使用者指出可研究使用另一個既有依賴來源。

外部來源故障時先檢查是否有符合版本、內容身分與驗證強度的等價取得方式。不能把某個 resolver 當成唯一可能路徑，也不能把換來源當成略過驗證。來源沒有替代來源成功或交付完成的證據。
