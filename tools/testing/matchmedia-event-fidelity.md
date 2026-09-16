---
title: "matchMedia mock 要同步狀態與事件入口"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 4bf995dfdf7b4bfd7c664e1e9c48b623a44c0eb6d211691fbba61e036171297b
---

# matchMedia mock 要同步狀態與事件入口

歷史助手發現 mock 的 matches 固定不變，hook 讀取不到模擬切換；另一測試直接呼叫 listener，繞過負責更新 currentMatches 的 dispatchEvent。

測試媒體查詢變更時，讓狀態與事件派送共同遵循實際契約。直接叫 callback 不一定等價於發生一個 change 事件，需驗證 hook 看到的狀態。來源只有測試修正診斷，沒有產品修復通過的結果；同名按鈕選擇器問題另計。
