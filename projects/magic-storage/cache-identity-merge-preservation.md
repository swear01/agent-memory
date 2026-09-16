---
title: "衝突解析要同時保留 cache 身份與 classifier 契約"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: b434bf11b98742dc8a7f576f3e19cf1c42a673a6e3af74e43540d6bd94ac01ca
---

# 衝突解析要同時保留 cache 身份與 classifier 契約

歷史只讀 merge-tree 指出兩個 PR 在 selected mod ID、inspectable cache 與 migration 規則衝突；同一 jar 不同 mod ID 需隔離，舊 classifier cache 的測試又必須保留 stale 版本。

按行為整合衝突，不能整檔選 ours／theirs。分開 current classifier 版本與故意舊版 fixture，沿 scan、validate、migration、diff 等 consumer 核對身份；migration 例外不能流入正常驗證。來源僅模擬與計畫，實際合併 head 若改變仍需重驗，沒有 rebase 後通過證據。
