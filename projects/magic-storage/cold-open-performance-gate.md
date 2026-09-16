---
title: "First open 效能不能在 fixture 預熱後量測"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 759b0e5ab1c393e09f2bf3e07ee431524d5853bbc9608e902ff17a1c396c65cf
---

# First open 效能不能在 fixture 預熱後量測

歷史助手發現 30k 報告的 first open 在 fixture 預熱全部索引後量測，因而把預熱後 2ms 當成第一次開啟，無法覆蓋使用者冷開卡頓。

將冷開與 warm 操作分開，讓 first-open gate 從符合真實情境的未預熱狀態開始，記錄量測前置条件。不要用預熱數字替代冷開驗收。來源只有漏洞定位及修改 gate 的計畫，沒有修正後測量結果。
