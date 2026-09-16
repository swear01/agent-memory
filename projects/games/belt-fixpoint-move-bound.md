---
title: "輸送帶 fixpoint 需遵守每 tick 的移動上限"
scope: projects/games
status: active
updated: 2026-09-16
evidence_digest: 2bd677e1cbff68c43dd78eb851eb60348d33cacebf52e180342b8ffde90e690e
---

# 輸送帶 fixpoint 需遵守每 tick 的移動上限

歷史助手推導帶一個空位的環狀輸送帶會使 while(movedThisPass) 持續成立：同一單位可跨多輪在同一 tick 反覆移動，於是提出 movedThisTick guard。

依遊戲契約限制每單位每 tick 可移動次數，讓迴圈有有限進展界線；檢查新增單位是在 MOVE 前後加入，避免追蹤陣列不一致。來源只到推導、修改提案及準備重跑，沒有 timeout 後的成功測試，不把暫無輸出本身當成無限迴圈證據。
