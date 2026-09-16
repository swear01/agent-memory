---
title: "卸載後的 guard 不能使用過期 route 快照"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: df55447f11fb4e7f04af13c08c91d69ceefc030a428ba770f7bda781de49dabb
---

# 卸載後的 guard 不能使用過期 route 快照

歷史 review 指出元件卸載後，closure 裡 render-time pathname 已過期；助手同意需要讀 router 的即時狀態，並查找測試 harness。

針對晚到 callback 或卸載後 guard，明確區分事件發生時的快照與判定時的現況。若判斷依賴目前路由，就讀適用的 live state，並測導航後 callback 到達的情境。不是要求所有 effect 都忽略原始快照；來源沒有修補後測試結果。
