---
title: "Transform 隔離須驗證解析後的 runtime classpath"
scope: tools/gradle
status: active
updated: 2026-09-16
evidence_digest: f6eb7460c5883831dfc9f9c85284308e330b49a93b76f0be54349b6cdcf28e2e
---

# Transform 隔離須驗證解析後的 runtime classpath

保存的 review 指出 harness 只看直接 runtimeOnly，漏掉 implementation、其他繼承 configuration 與 transitive dependency；審查者回報可讓 pristine 與 transformed jar 同時出現在 runtime classpath。

檢查 finalized dependency graph 與實際解析產物，確認預期 transformed jar 的數量及 pristine 副本是否被排除。只 grep build.gradle 或測 helper map 不足以驗證 Gradle runtime wiring。

來源是審查報告及測試建議，沒有修後執行證據。版本綁定與是否需要 transform 仍以專案契約為準。
