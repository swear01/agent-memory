---
title: "CI 的隱含建置模式可能改變 ancestry 產物"
scope: tools/gradle
status: active
updated: 2026-09-16
evidence_digest: 52a2db41ee6dbe24488cdd45d0a4c1b585c99262c8cddaf2ee149e05bef7173d
---

# CI 的隱含建置模式可能改變 ancestry 產物

歷史助手與控制者交接均指出，ModDevGradle 的 CI 模式啟用了 disableRecompilation，本機 audit 卻使用重編譯產物，兩者 SHA 不同，exact ancestry gate 因此拒絕。

將影響產物的模式明確固定，再於同一模式產生 audit 與 contract；保留舊產物作為差異證據，不能直接放寬雜湊 gate。來源只有固定旗標、重產與驗證的計畫，沒有完整通過結果，也不把該歷史預設外推到所有版本。
