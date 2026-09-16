---
title: "Translate 的 delta 不等於一路滑到牆前"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 96181539a60ae6d0cf08792caccde8b7bbec3cf0ab635bb686a74fed2dc8a71e
---

# Translate 的 delta 不等於一路滑到牆前

助手原先按滑到牆前推算位置，後來承認 push 的目標是起點加 delta：無阻擋時 push 移動一格、push2 移動兩格。原文另外區分遇牆停止與遇 hazard 失敗。

由 catalog、目標座標與逐格 sweep 的實際停止條件推算移動，再以真實 solver 驗證產生的關卡；不能把不同障礙物都簡化成停止移動。

這是當時的診斷自述。草稿的通用難度公式、所有地圖路徑相同及 generator 已正確等結論不採用，來源沒有完整端到端驗證。
