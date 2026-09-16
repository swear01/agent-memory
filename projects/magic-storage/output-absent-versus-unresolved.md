---
title: "無法解析的 item output 不可視為不存在"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 6a121eb106cba5f3a5a6278adb7d75ee5c3caa0c66b7aeef1cb822e871869153
---

# 無法解析的 item output 不可視為不存在

保存的 read-only audit 指出 Drying 共同 helper 遇 ItemStackFromIngredient 時解析成空，若另有 exact fluid output 便接受配方，把 fluid 誤升為 primary；兩種 Basin family 共用該路徑。

在共同邊界分開 raw output absent、存在但不支援、以及精確非空解析成功。只有契約允許且 item 真正不存在時才能選 fluid primary，兩個 caller 都要驗證拒收。來源只是審查及建議，未執行 GameTest；靜態證據缺口和檔尾空行是獨立發現，不算此語意問題已修。
