---
title: "找回可用 netlist 不等於能重建 synthesis 流程"
scope: domains/atpg
status: active
updated: 2026-09-16
evidence_digest: 2dcbc83aca5011aad9067284f7077e36175e27cf40a3091cd1473eebb42146ee
---

# 找回可用 netlist 不等於能重建 synthesis 流程

歷史助手自述找回的 netlist 與現有 mask 名稱相符且 ATPG 可跑，隨後明確承認只是恢復工作產物，尚未重建可重現的 synthesis 來源鏈。

分別交付 artifact 身分、目前相容性與可重生流程。名稱吻合和跑完不能代替固定工具版本、輸入及生成命令。來源另有錯誤 cwd 使工具等輸入的執行問題，不能混為 synthesis 根因；完整重建仍是後續計畫。
