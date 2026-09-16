---
title: "只測 helper 不能證明公開流程仍會呼叫它"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 527c6c5e152262a82ee82cfafe2bf32605c9eefe1e99f68f668db24d5db26f1e
---

# 只測 helper 不能證明公開流程仍會呼叫它

同次 review 指出死亡測試直接呼叫 CancelAllAbilities，未從致死流程進入；移除 Respawn 的呼叫也能綠。另有只清 HashSet、沒有驗證 IgnoreCollision 恢復的測試。

對宣稱的整合行為從真正公開入口觸發，並檢查可觀察副作用；helper 的直接測試保留其局部範圍，不稱完整流程回歸。

來源為唯讀測試審查，没有新的整合測試通過。這些測試缺口不證明當時 production 必定已壞。
