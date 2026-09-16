---
title: "Java generator 不能直接使用 JVM binary class name"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 37119240624b0d0d5aa12b9f0c47dd879c54ca7625e990b62f281c9903bdb204
---

# Java generator 不能直接使用 JVM binary class name

歷史審查指出 scanner 與 contract 保留 JVM binary name，generator 卻直接當 Java source type 使用；建議另存 source_class，依 InnerClasses owner metadata 推導具名巢狀型別。

保留 binary 身份與 source 名稱兩者，驗證 audit／contract 對應，再涵蓋所有生成 shape。不能全域把美元符號換成點，合法 top-level 名稱仍可能含美元符號。來源只跑兩個 scanner 測試，沒有 generator regression 或 migration 完成證據；anonymous owner 的可定址性也需另行處理。
