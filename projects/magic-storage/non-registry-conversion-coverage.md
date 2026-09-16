---
title: "RecipeManager 外的轉換機不可默默漏掉"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: aeed18f1922d94b0f9e5c8879ed7068f8b7f1ff16c8c98d3e2bdb37eb3776666
---

# RecipeManager 外的轉換機不可默默漏掉

使用者指出 acceptance 中的 Nutritional Liquifier 並非 RecipeManager recipe type，而是由 TileEntity 依 food input 合成轉換資料。只列舉 recipe registry 可能漏掉已承諾支援的機器。

先確認轉換來源是否符合既有 exact/server contract。能安全支援時以 fixture 證明；不能時保留精確 source/API 阻擋，並在範圍文件明示不支援，不可靜默略過。來源是使用者的要求，未證明實作可行或已完成。
