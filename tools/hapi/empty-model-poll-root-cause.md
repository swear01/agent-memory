---
title: "空 model list 的輪詢上限不能代替根因調查"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 9e9bd0678acf2cc9e5a0dbc9bbee833b70a5c08ce877270d87983bfd12ef1c67
---

# 空 model list 的輪詢上限不能代替根因調查

歷史助手轉述 bot finding：backend 持續成功回傳空 model list 時，前端每秒輪詢；它提出十次上限，使用者追問根本問題是為何拿不到清單。

為空結果與錯誤設有界輪詢，同時沿 backend 的模型取得路徑保留失敗原因。上限可限制資源消耗，不能記成清單已恢復；固定十次僅為當次提議，不是通用門檻。來源沒有程式修改、測試或根因結論。
