---
title: "清理測試須涵蓋根目錄內的 symlink 與刪除結果"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: ac71d82c52e7cd59ee10ae5f20c907bd0180fcd1ce4605a7dc86e7ef28c31151
---

# 清理測試須涵蓋根目錄內的 symlink 與刪除結果

歷史 review 指出清理只檢查 run 與 world 根路徑，未覆蓋巢狀 symlink，並忽略 deleteDir 的回傳值；既有測試只比對來源字串。

在實際使用的刪除 API 上驗證不跟隨連結、外部 sentinel 保留及失敗回傳，不能只靠頂層路徑或字串存在推論安全。來源是審查發現與測試建議，沒有重跑庫行為或修復證據；另一項 addon contract 矩陣缺口屬獨立問題，未合併為同一原因。
