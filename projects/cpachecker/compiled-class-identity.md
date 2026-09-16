---
title: "舊 classes 的執行不能驗證新 Java patch"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 14e23ec4901c2c75337800844044306d28262c5f46ff29cfd6c027c56d82399a
---

# 舊 classes 的執行不能驗證新 Java patch

歷史助手在缺少 ant 時先用現有 CPAchecker 跑 smoke，後承認 scripts/cpa.sh 仍使用舊 compiled classes，新 patch 尚未進入執行產物。

驗證 Java 原始碼變更前先確認建置工具與實際使用的 class 產物；既有 binary 能啟動不等於新來源已編譯。執行耗時本身也不能證明 classes 新舊，仍需建置與載入身分證據。來源沒有重編完成或 fallback 修復結果。
