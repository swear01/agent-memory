---
title: "檔案存在不代表實際建置已使用該 artifact"
scope: "tools/build-verification"
status: active
updated: 2026-09-07
evidence_digest: 6b7589abdc01ae0506893c1c4143dc9f5a398670f12e4806ce297481f856d5fa
---

# 檔案存在不代表實際建置已使用該 artifact

## Problem

歷史 audit 列出 expected 92、disk found 92，但 manifest exact 80，並逐項列出未被 manifest 引用的檔案。

## Durable rule

核對目標 artifact 的雜湊與實際 build manifest/classpath 歸屬；磁碟清單齊全不能代替建置輸入證據，也不能據此直接刪除未引用檔案。

## Boundary

限此歷史案例及相同條件；不能覆蓋目前任務授權或最新專案規則。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
