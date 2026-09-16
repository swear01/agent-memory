---
title: "使用者要從 PR 擴展 bug list 時不要改成合成注入引擎"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: d5524db25e8ee0a710baa29936be5ecac6aa17d6f5ba35ef3028bc5ceb5d17e4
---

# 使用者要從 PR 擴展 bug list 時不要改成合成注入引擎

使用者明確要求從 PR 擴展類似 Encarsia 的 bug list；歷史助手承認又把方向做成系統性合成注入，保存的計畫仍以枚舉 mux／driver 為主要擴展來源。

先對齊要擴展的來源、分類與確認標準，再設計收集流程。採用相近故障風格不代表真實歷史 bug 或同等品質；靜態候選數也不是 confirmed 產量。來源只到更正方向與計畫文字，沒有 PR-driven 引擎或資料驗收完成證據。
