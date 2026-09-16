---
title: "Descriptor 生成前須拒絕跨欄位重複 dependency"
scope: tools/gradle
status: active
updated: 2026-09-16
evidence_digest: f6eb7460c5883831dfc9f9c85284308e330b49a93b76f0be54349b6cdcf28e2e
---

# Descriptor 生成前須拒絕跨欄位重複 dependency

同一份歷史 review 指出 target.dependency 可再次出現在 runtime_dependencies，generator 又把 target 前置加入，產出被 Gradle 拒絕的重複 descriptor；原 CLI/schema 檢查未攔截。

在整份 contract 的欄位關係上驗證唯一性，並把通過 schema 的輸入交給真正 consumer 驗證。單一欄位各自合法不代表組合後合法。

這與解析後 classpath 的 pristine 污染是不同缺陷；本次只保留審查所述機制，不宣稱修正或 regression test 已完成。
