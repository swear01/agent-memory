---
title: "Dev smoke 要遵守指定主機與資源範圍"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: a948529db49ca5c909a55b9eed6da3dbdee6013c9cb414bb697397c0331e5294
---

# Dev smoke 要遵守指定主機與資源範圍

使用者明確要求不要在小型正式服務主機跑 HAPI dev smoke，應在指定開發機執行，並指出 Swear Review 是正式服務，不是可清掉的殘留。

在啟動入口落實適用主機與資源限制，分清正式服務、本次測試及其擁有的程序。Ad-hoc detached harness 的清理與 dev hub 曝露另需獨立處理；不能以轉移 build 就稱它們一併解決。來源只有加入維護 harness 與限流的提案，沒有落地驗證。
