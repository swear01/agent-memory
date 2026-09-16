---
title: "Fork schema 版本較新仍可能缺少上游資料表"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 637e5a880e92c27fc8354ec0844eb5a333224af1361c25bfcc5954dc73772fc3
---

# Fork schema 版本較新仍可能缺少上游資料表

歷史助手回報 Hub DB 顯示 V27，卻缺 events 與 event_links，新 binary 因此拒啟動；並將缺口歸因於 fork 版本血統跳過上游的較早 migration。

更新前核對實際 schema 物件與來源血統，不能只比較 user_version。補償 migration 必須依完整 schema 契約與可恢復備份驗證；手動建表計畫不是驗證結果。片段中的 handoff 迴圈與背景測試目錄錯誤是另外兩個問題，不把它們併成同一根因。
