---
title: "Gradle 集合取值須符合實際 DSL 與型別"
scope: tools/gradle
status: active
updated: 2026-09-16
evidence_digest: 5cd17df9fc3f441729ff80ca1b438f9cf4b47deb3ec7978c82ad39c27b3c078d
---

# Gradle 集合取值須符合實際 DSL 與型別

使用者提供的 live retry 回報指出，SHA task 已執行到 build.gradle，卻因 LinkedHashSet.single() 不存在而失敗。回報同時說 configuration cache 已能重用；這是集合 API 錯誤，不能因此把快取機制也判為失敗。

使用實際 Gradle DSL 與集合型別支援的方法。此處先驗證 artifacts.size() 恰為 1，再以 iterator().next() 取得 artifact 並讀取內容；不要把別種語言的集合擴充方法直接搬進目前 DSL。

來源只有該次使用者回報與建議寫法，未包含修正後 task 通過的證據，也不將 single() 不存在的結論擴張到所有語言或 DSL。
