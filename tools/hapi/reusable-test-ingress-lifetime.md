---
title: "清理計畫要分開一次性程序與長期測試資源"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 8ca80f1a843bb3639428d2d357f7529b59067c1daaa3527fce4fd84ed1c02ef1
---

# 清理計畫要分開一次性程序與長期測試資源

助手在 clone-local 維運筆記加入測完刪除雲端 ingress rule 的步驟；使用者更正同一固定測試 port 下次還要用，該 rule 不需刪除。

依資源的已授權生命週期制定 cleanup。一次性工作程序結束不代表共用測試入口也要移除；明確記錄保留理由與作用範圍。此要求限當時指定測試資源，不是永久保留所有開放埠；來源沒有筆記更正完成的證據。
