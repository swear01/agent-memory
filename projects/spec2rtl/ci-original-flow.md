---
title: Spec2RTL CI must preserve the original project flow
scope: projects/spec2rtl
status: active
updated: 2026-10-10
---

# CI 的流程邊界

使用者明確要求：CI 必須先完整執行原本 Spec2RTL 的 orchestrator、skills 與工具流程，之後再檢查生成 RTL 的正確性與 PPA。被評估的就是原本流程，CI 不得自行改造它。

- 保留原本的步驟順序、必要產物、驗證、除錯與失敗處理規則；原本流程允許的並行仍可使用。
- 外部正確性檢查與 PPA 是流程完成後的驗收，不得為縮短時間自行提前插入中途、將結果回饋成新的 agent 修復流程，或取代原本的 testbench/assertions/alignment。
- 不得藉 CI 優化擅自修改 skills、提前 alignment/assertions、刪減必要步驟或增加原本沒有的設計任務。若需要改變原本流程，必須作為獨立提案，另取得使用者明確授權。
- 縮時應在保留原本流程的前提下評估資源、SDK 開銷、已授權的固定 baseline 快取與 benchmark 大小；不能只為達到時間目標就改變評估內容。
- 先前提出的「在 RTL/lint 後提前執行獨立 VCS 檢查」方案已被使用者否決，不得沿用為待執行計畫。

此要求由使用者於 2026-10-10 直接重申，並要求加入共享記憶；不是由 CI 失敗或模型推論自行得出的政策。
