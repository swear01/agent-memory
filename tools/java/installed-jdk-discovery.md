---
title: "建置找不到 Java 時先查已安裝 JDK"
scope: tools/java
status: active
updated: 2026-09-16
evidence_digest: e8d6709cb0271602e857e2d439466f12394273db90d946ef0f1f1a28f3d693cc
---

# 建置找不到 Java 時先查已安裝 JDK

歷史助手誤以為需要另裝 Java，後承認本機已有 Homebrew keg-only JDK，只是沒有進入 shell 的 JAVA_HOME/PATH；使用者要求直接完成設定。

先核對已有工具鏈、專案要求與實際 shell 環境，再安裝或更換。keg-only 不等於未安裝，JRE 也不等於具備建置用 JDK。來源包含助手自述使用既有 JDK compileJava 成功，但未附可獨立核對輸出，也未證明永久 shell 設定完成。
