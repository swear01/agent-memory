---
title: "測試不能靠 TearDown 保證正式資產不被污染"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: e72e524e182750ff82d1f047bb82d6b382122e58ce4aa8e7551f9a26d0ae6186
---

# 測試不能靠 TearDown 保證正式資產不被污染

歷史唯讀 review 指出測試把正式 AudioManager Prefab 改綁 TestOnly mixer，再於 TearDown 回寫 YAML；Editor 崩潰或 Domain Reload 會留下改過的資產。

改用測試副本或明確受限的測試資產入口，驗證操作範圍與清理結果。TearDown 是正常收尾流程，不能當所有中斷情況都能復原的保證。

來源提出缺陷與修法，沒有測試隔離修正完成證據；其他 discovery、CI 與 clip 問題不併成這項根因。
